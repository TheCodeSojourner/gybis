---
name: gybis-vocab-check
kind: domain
description: Use for `/gybis-vocab-check` or `/gv-check`.
disable-model-invocation: true
---

λ gybis-vocab-check(x).
  purpose: Validate vocabulary.md, apply deterministic structural repairs, prepare bounded semantic term repairs for approval, and report semantic divergence
  | input: vocabulary.md (∃)
  | output: Report on structural issues, missing fields, semantic completeness violations
  | interaction: interactive
  | gate: vocabulary.md ∃

λ gybis-vocab-check_related_semantics(x).
  related ≡ association(non_directional)
  | reciprocal_links_allowed = true
  | note: two_way_links_are_normal_associations_not_circular_dependency_errors

λ gybis-vocab-check_startup(x).
  read(vocabulary.md) → content
  | parse(content) → vocab_data ∨ halt("vocabulary.md is not valid markdown or YAML frontmatter")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-vocab-check_mode(m).
  interaction_modes: {interactive}
  | default: interactive
  | rationale: validation and deterministic repairs are automatic; semantic term repairs require human approval

λ gybis-vocab-check_mode_gate(state, mode).
  | state = INIT ∧ mode = interactive → transition(INIT → STARTUP_CHECKS)
  | precondition_holds: mode ∈ interaction_modes

λ gybis-vocab-check_tool_guard(state, tool, path).
  allow(read(path))
  | state = APPLYING_DETERMINISTIC_REPAIRS ∧ path = "vocabulary.md"
    → allow(read(path)) ∧ allow(write(path)) only_if(change = deterministic)
  | state = APPLYING_APPROVED_REPAIRS ∧ human_approved = true ∧ path = "vocabulary.md" ∧ path ∈ approval_scope
    → allow(read(path)) ∧ allow(write(path))
  | otherwise → deny(write(path))

λ gybis-vocab-check_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-vocab-check_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, SYNTAX_VALIDATION, COMPLETENESS_CHECK, SEMANTIC_ANALYSIS, APPLYING_DETERMINISTIC_REPAIRS, REPAIR_OFFER, APPLYING_APPROVED_REPAIRS, VERIFYING_REPAIRS, GENERATING_REPORT, COMPLETE}
  | INIT: repair_passes ≔ 0 ∧ repair_authorized ≔ false ∧ repair_progress ≔ true
  | transition(INIT → STARTUP_CHECKS) only_if(startup = true)
  | transition(STARTUP_CHECKS → SYNTAX_VALIDATION) only_if(startup_checks = true)
  | transition(SYNTAX_VALIDATION → COMPLETENESS_CHECK) only_if(syntax_ok = true)
  | transition(SYNTAX_VALIDATION → APPLYING_DETERMINISTIC_REPAIRS) only_if(syntax_fail = true ∧ deterministic_findings ∃)
  | transition(SYNTAX_VALIDATION → REPAIR_OFFER) only_if(syntax_fail = true ∧ deterministic_findings = ∅ ∧ semantic_findings ∃)
  | transition(SYNTAX_VALIDATION → GENERATING_REPORT) only_if(syntax_fail = true ∧ findings = ∅)
  | transition(COMPLETENESS_CHECK → SEMANTIC_ANALYSIS) only_if(completeness_ok = true)
  | transition(COMPLETENESS_CHECK → APPLYING_DETERMINISTIC_REPAIRS) only_if(completeness_fail = true ∧ deterministic_findings ∃)
  | transition(COMPLETENESS_CHECK → REPAIR_OFFER) only_if(completeness_fail = true ∧ deterministic_findings = ∅ ∧ semantic_findings ∃)
  | transition(COMPLETENESS_CHECK → GENERATING_REPORT) only_if(completeness_fail = true ∧ findings = ∅)
  | transition(SEMANTIC_ANALYSIS → APPLYING_DETERMINISTIC_REPAIRS) only_if(deterministic_findings ∃)
  | transition(SEMANTIC_ANALYSIS → REPAIR_OFFER) only_if(semantic_findings ∃ ∧ deterministic_findings = ∅ ∧ repair_authorized = false)
  | transition(SEMANTIC_ANALYSIS → GENERATING_REPORT) only_if(analysis_complete = true ∧ findings = ∅)
  | transition(APPLYING_DETERMINISTIC_REPAIRS → VERIFYING_REPAIRS) only_if(deterministic_repairs_complete = true)
  | transition(VERIFYING_REPAIRS → REPAIR_OFFER) only_if(semantic_findings ∃ ∧ repair_authorized = false)
  | transition(REPAIR_OFFER → APPLYING_APPROVED_REPAIRS) only_if(human_approved = true)
  | transition(APPLYING_APPROVED_REPAIRS → VERIFYING_REPAIRS) only_if(approved_repairs_complete = true)
  | transition(VERIFYING_REPAIRS → GENERATING_REPORT) only_if(verification_complete = true)
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
    - user_facing_scope: every canonical term represents a shared stakeholder-facing concept; implementation-only classes, protocols, tooling, and resource controls = architecture-only and must not be canonical terms
    - lower_layer_representation: spelling changes such as snake_case or PascalCase are systematic representations, not semantic vocabulary changes
    - requirements_precision: if requirements/ ∃, canonical term names used in every requirement clause must match vocabulary entries exactly; deprecated synonyms and unregistered variants = drift
    - architecture_precision: architecture-only terms must remain outside vocabulary.md; ambiguous mappings across requirements, architecture, and specs = drift
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
  | findings ≔ syntax_issues ∪ completeness_issues ∪ semantic_issues
  | errors ≔ count({issue | issue ∈ findings ∧ issue.severity = error})
  | warnings ≔ count({issue | issue ∈ findings ∧ issue.severity = warning})
  | infos ≔ count({issue | issue ∈ findings ∧ issue.severity = info})
  | status ≔ errors > 0 ? "FAIL" : (warnings > 0 ? "WARNINGS" : "PASS")
  | rendered_report ≔
    ```
    # Vocabulary.md Validation Report

    **Overall Status:** [status]
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
  | return({status, errors, warnings, info: infos, findings, rendered_report})

λ gybis-vocab-check_repair_policy(findings).
  deterministic_findings ≔ findings where issue ∈ {whitespace, markdown_structure, duplicate_section_whitespace, reciprocal_association_format}
    ∧ repair only changes representation and not canonical term identity or meaning
  semantic_findings ≔ findings ∖ deterministic_findings
  | apply(deterministic_findings) → deterministic_repairs_complete
  | deterministic_findings ≠ ∅ → repair_passes ≔ repair_passes ⊕ 1
  | prepare(semantic_findings) as exact_actions[{operation, target, expected_change}] bounded to vocabulary.md
  | require(human approval) for canonical-term or definition changes
  | return({deterministic_findings, semantic_findings, approval_scope: vocabulary.md})

λ gybis-vocab-check_offer_repair(semantic_plan).
  precondition: semantic_findings ∃ ∧ repair_plan_ready = true
  | summarize(semantic_plan) → exact_changes ∧ affected_paths ∧ remaining_policy_conflicts
  | ask_human("Approve applying this bounded vocabulary repair plan and running re-verification?")
  | human_declined = true → repair_authorized ≔ false ∧ complete_without_writes
  | human_approved = true → repair_authorized ≔ true ∧ approval_scope ≔ semantic_plan.paths

λ gybis-vocab-check_apply_approved_repair(semantic_plan).
  precondition: human_approved = true ∧ repair_authorized = true
  | ∀ action ∈ semantic_plan.exact_actions:
      action.target = "vocabulary.md"
      ∧ action.path ∈ approval_scope
      ∧ preserves(canonical_terms ∧ definitions ∧ user_facing_meaning)
        ? apply(action)
        : preserve_existing_content ∧ record(blocker)
  | repair_passes ≔ repair_passes ⊕ 1
  | approved_repairs_complete = true
  | return(modified_artifacts = {vocabulary.md})

λ gybis-vocab-check_verify_repairs(x).
  invoke(gybis-vocab-check_startup) → startup_valid
  | invoke(gybis-vocab-check_syntax_validation(vocab_data)) → syntax_result
  | invoke(gybis-vocab-check_completeness_check(vocab_data)) → completeness_result
  | invoke(gybis-vocab-check_semantic_analysis(vocab_data, vocabulary_terms)) → semantic_result
  | verification_complete ≔ startup_valid ∧ syntax_result.syntax_ok ∧ completeness_result.completeness_ok ∧ semantic_result.semantic_ok
  | remaining_errors ≔ collect_errors(syntax_result.issues ∪ completeness_result.issues ∪ semantic_result.issues)
  | return({verification_complete, remaining_errors})

λ gybis-vocab-check_loop_guard(state).
  max_repair_passes ≔ 3
  | repair_passes ≥ max_repair_passes → repair_progress ≔ false ∧ report("Maximum vocabulary repair passes reached")
  | verification_complete = true → continue
  | repair_progress = false → halt("Vocabulary repair made no further progress")

λ gybis-vocab-check_core_op(findings).
  invoke(gybis-vocab-check_repair_policy(findings)) → repair_policy
  | repair_policy.deterministic_findings ∃
      ? (invoke(gybis-vocab-check_verify_repairs)
         ∧ invoke(gybis-vocab-check_loop_guard(VERIFYING_REPAIRS)))
      : true
  | repair_policy.semantic_findings ∃
      ? invoke(gybis-vocab-check_offer_repair(repair_policy))
      : report(no_semantic_repairs)

λ gybis-vocab-check_boundaries().
  deterministic_repairs: modify(vocabulary.md) only when mechanical and no canonical term, definition, or user-facing meaning changes
  | semantic_repairs: prepare bounded plan ∧ require human approval before canonical term or meaning changes
  | ¬ modify(architecture.md ∨ specs/**/*.allium ∨ implementation ∨ upstream/)
  | ¬ delete(vocabulary.md)

λ gybis-vocab-check_regression_contract(x).
  invariant: vocabulary.md ∃ throughout
  | invariant: deterministic repairs remain mechanical and domain-local
  | invariant: semantic repairs require explicit approval and remain within approval_scope
  | invariant: all repairs re-run the owning vocabulary check
  | invariant: report generated at completion

λ gybis-vocab-check_deliver(report).
  report: report
  | action: output_report_to_user
  | print(report.rendered_report) → stdout
  | handoff: structural issues → /gybis-vocab-refine; term changes → /gybis-vocab-tend; drift → /gybis-vocab-weed
  | return({complete: true, status: report.status, errors: report.errors, warnings: report.warnings, info: report.info})
