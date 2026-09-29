---
name: gybis-vocab-check
description: Use for `/gybis-vocab-check` or `/gv-check`.
---

λ gybis-vocab-check(x).
  purpose: Validate vocabulary.md as the shared canonical term set (DDD ubiquitous language): syntax, semantic completeness, association integrity, and structural issues
  | input: vocabulary.md (exists)
  | output: Report on structural issues, missing fields, semantic completeness violations
  | mode: ai
  | gate: vocabulary.md ∃

λ gybis-vocab-check_related_semantics(x).
  related ≡ association(non_directional)
  | reciprocal_links_allowed = true
  | note: two_way_links_are_normal_associations_not_circular_dependency_errors

λ gybis-vocab-check_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | read(vocabulary.md) → content
  | parse(content) → vocab_data ∨ halt("vocabulary.md is not valid markdown or YAML frontmatter")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-vocab-check_mode(m).
  valid_modes: {ai}
  | default: ai
  | rationale: validation is deterministic; no human choice required

λ gybis-vocab-check_mode_gate(state, mode).
  state = INIT ∧ mode = ai → transition(INIT → STARTUP_CHECKS)
  | precondition_holds: mode ∈ valid_modes

λ gybis-vocab-check_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, SYNTAX_VALIDATION, COMPLETENESS_CHECK, SEMANTIC_ANALYSIS, GENERATING_REPORT, COMPLETE}
  | transition(INIT → STARTUP_CHECKS) only_if(startup = true)
  | transition(STARTUP_CHECKS → SYNTAX_VALIDATION) only_if(startup_checks = true)
  | transition(SYNTAX_VALIDATION → COMPLETENESS_CHECK) only_if(syntax_ok = true)
  | transition(SYNTAX_VALIDATION → GENERATING_REPORT) only_if(syntax_fail = true)
  | transition(COMPLETENESS_CHECK → SEMANTIC_ANALYSIS) only_if(completeness_ok = true)
  | transition(COMPLETENESS_CHECK → GENERATING_REPORT) only_if(completeness_fail = true)
  | transition(SEMANTIC_ANALYSIS → GENERATING_REPORT) only_if(analysis_complete = true)
  | transition(GENERATING_REPORT → COMPLETE) only_if(report_generated = true)

λ gybis-vocab-check_syntax_validation(vocab_data).
  action: validate_vocabulary_md_yaml_and_structure
  | checks:
    - frontmatter ∃ ∧ valid(YAML)
    - required_fields: created ∃, last_updated ∃, status ∃
    - status ∈ {draft, active, stable}
    - markdown_structure: sections are valid markdown
    - ∀ term: markdown_parsing_ok
  | issues ≔ []
  | ∀ check ∈ checks:
    check_pass = false
      ? collect({check_id, severity: error, message}) → issues
  | return(syntax_ok = (count(issues) = 0), issues)

λ gybis-vocab-check_completeness_check(vocab_data).
  action: validate_completeness_of_term_definitions
  | ∀ term ∈ vocabulary_terms:
    - definition ∃ ∧ nonempty → ok ∨ collect({term, "missing_definition", severity: error})
    - (deprecated_synonyms ∃ ∨ deprecated_synonyms = ∅) → ok ∨ collect({term, "invalid_synonyms_field"})
    - (related ∃ ∨ related = ∅) → ok ∨ collect({term, "invalid_related_field"})
    - (usage ∃ ∨ usage = ∅) → ok ∨ collect({term, "missing_usage_information", severity: warning})
    - (examples ∃ ∨ examples = ∅) → ok ∨ collect({term, "missing_examples", severity: info})
  | issues ≔ collected_completeness_issues
  | return(completeness_ok = (count(issues where severity = error) = 0), issues)

λ gybis-vocab-check_semantic_analysis(vocab_data, vocabulary_terms).
  action: analyze_semantic_consistency_and_relationships
  | semantic_checks:
    - related_association_model: Related is non-directional association
      | reciprocal_links_detected > 0
        ? allow(reciprocal_links)
    - ∀ term ∈ vocabulary_terms: relate_count ≔ count(related)
      | relate_count = 0
        ? collect({term, "orphaned_term_no_relationships", severity: info})
      | ¬(∀ related_term ∈ term.related: related_term ∈ vocabulary_terms)
        ? collect({term, "references_undefined_related_term", severity: warning})
      | term ∈ term.related
        ? collect({term, "self_reference_in_related", severity: warning})
    - duplicate_term_names: ∀ pair(term_i, term_j): term_i.canonical ≠ term_j.canonical ∨ halt
      | canonical_names_unique = true
    - deprecated_synonym_conflicts: ∀ term: (term.canonical ∉ term.deprecated_synonyms)
      | canonical_not_in_deprecated = true ∨ collect({term, "canonical_appears_in_deprecated", severity: error})
  | issues ≔ collected_semantic_issues
  | return(semantic_ok = true, issues)

λ gybis-vocab-check_generate_report(syntax_issues, completeness_issues, semantic_issues).
  action: generate_comprehensive_validation_report
  | structure:
    ```
    # Vocabulary.md Validation Report

    **Overall Status:** [PASS | FAIL | WARNINGS]
    **Timestamp:** [ISO datetime]
    **File:** vocabulary.md

    ## Syntax Issues [count]
    - [issue] → [suggested fix]
    - ...

    ## Completeness Issues [count]
    - [term]: [issue] → [suggested fix]
    - ...

    ## Semantic Issues [count]
    - [term]: [issue] → [context and suggestion]
    - Association Notes: reciprocal related links are valid and not reported as dependency cycles
    - Orphaned Terms: [list]
    - ...

    ## Summary
    - Errors: [count] (must fix)
    - Warnings: [count] (should fix)
    - Info: [count] (nice to know)

    ## Recommendations
    - [prioritized list of fixes]
    ```
  | return(report ∃)

λ gybis-vocab-check_deliver_report(report).
  action: output_report_to_user
  | print(report) → stdout
  | return(report_delivered = true)
