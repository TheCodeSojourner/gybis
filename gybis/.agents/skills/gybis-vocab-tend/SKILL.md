---
name: gybis-vocab-tend
description: Use for `/gybis-vocab-tend` or `/gv-tend`.
---

λ gybis-vocab-tend(x).
  purpose: Evolve vocabulary.md as the shared canonical term set (DDD ubiquitous language) with human feedback while maintaining consistency across architecture.md and specs/**/*.allium
  | input: vocabulary.md ∃
  | output: vocabulary.md evolved with approved changes; affected specs/architecture optionally rewritten
  | mode: interactive
  | gate: vocabulary.md ∃

λ gybis-vocab-tend_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | verify(vocabulary.md ∃) ∨ halt("vocabulary.md not found")
  | read(vocabulary.md) → vocab_content
  | parse(vocab_content) → vocabulary_terms
  | if(architecture.md ∃): read(architecture.md) → arch_content
  | if(specs/**/*.allium ∃): read_all_specs → spec_content
  | transition(INIT → STARTUP_CHECKS)

λ gybis-vocab-tend_mode(m).
  m ∈ {interactive}
  | default: interactive
  | mode_interactive: AI and developer collaborate on vocabulary evolution

λ gybis-vocab-tend_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {interactive} → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {interactive}) → halt("Invalid mode selection")

λ gybis-vocab-tend_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, ELICIT_FEEDBACK, ANALYZE_IMPACT, APPLY_CHANGES, VERIFY_CONSISTENCY, PROPAGATE_CHANGES, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → ELICIT_FEEDBACK) only_if(startup_checks = true)
  | transition(ELICIT_FEEDBACK → ANALYZE_IMPACT) only_if(developer_feedback ∃)
  | transition(ANALYZE_IMPACT → APPLY_CHANGES) only_if(impact_analysis ∃)
  | transition(APPLY_CHANGES → VERIFY_CONSISTENCY) only_if(changes_applied = true)
  | transition(VERIFY_CONSISTENCY → PROPAGATE_CHANGES) only_if(consistency = true)
  | transition(VERIFY_CONSISTENCY → ELICIT_FEEDBACK) only_if(consistency = false ∨ new_issues_detected = true)
  | transition(PROPAGATE_CHANGES → COMPLETE) only_if(propagation_complete = true ∨ propagation_skipped = true)

λ gybis-vocab-tend_tool_guard(state, tool, path).
  state = ELICIT_FEEDBACK ∨ state = ANALYZE_IMPACT
    → allow(read(path))
  | state = APPLY_CHANGES ∨ state = VERIFY_CONSISTENCY
    → allow(read(path)) ∧ allow(write(path)) only_if(path = vocabulary.md)
  | state = PROPAGATE_CHANGES
    → allow(read(path)) ∧ allow(write(path)) only_if(path ∈ {architecture.md} ∪ specs/ ∨ path = vocabulary.md)
  | ¬(state ∈ {ELICIT_FEEDBACK, ANALYZE_IMPACT, APPLY_CHANGES, VERIFY_CONSISTENCY, PROPAGATE_CHANGES})
    → deny(write(path))

λ gybis-vocab-tend_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-vocab-tend_elicit_feedback(vocabulary_terms).
  action: elicit_vocabulary_evolution_request
  | display: current_vocabulary_terms formatted as reference
  | prompt_strategy: ask what term(s) to modify, add, merge, or delete
  | sample_questions:
    - "Which term needs clarification? Should we rename it? Add/remove synonyms?"
    - "Are there new concepts that should be added to the vocabulary?"
    - "Should any terms be merged or split?"
    - "Do any definitions need revision?"
  | capture: user_response → feedback_request
  | output: feedback_request ∃ ∧ { add_term ∨ rename_term ∨ modify_definition ∨ merge_terms ∨ delete_term ∨ add_synonym ∨ remove_synonym }

λ gybis-vocab-tend_analyze_impact(feedback_request, vocabulary_terms, arch_content, spec_content).
  action: analyze_impact_of_vocabulary_changes
  | ∀ change ∈ feedback_request:
    change.type = rename
      ? (old_term ≔ change.old_name
         | new_term ≔ change.new_name
         | affected_in_arch ≔ search(arch_content, old_term) → occurrences
         | affected_in_specs ≔ search(spec_content, old_term) → occurrences
         | collect({change, affected_in_arch, affected_in_specs}) → impact_record)
      : (change.type ∈ {add, modify_definition, add_synonym}
         ? impact_record ≔ {change, affected = {}, requires_propagation = false}
         : (change.type ∈ {merge_terms, delete_term}
            ? report_potential_breakage ∧ ask_confirmation
            : impact_record ≔ {change, affected = {}}))
  | return(impact_analysis ∃)

λ gybis-vocab-tend_apply_changes(feedback_request, vocabulary_terms).
  action: apply_vocabulary_changes_to_vocabulary_md
  | ∀ change ∈ feedback_request:
    change.type = rename
      ? update_term_name(old, new) → modified_entry
      : (change.type = modify_definition
         ? update_definition(term, new_definition) → modified_entry
         : (change.type = add_term
            ? create_term_entry(new_term, definition, usage) → new_entry
            : (change.type ∈ {add_synonym, remove_synonym}
               ? update_synonyms(term, change) → modified_entry
               : skip)))
  | update_metadata: last_updated = now()
  | write(vocabulary.md) → persisted
  | return(changes_applied = true)

λ gybis-vocab-tend_verify_consistency(vocabulary_terms_updated, arch_content, spec_content).
  action: verify_vocabulary_consistency_with_architecture_and_specs
  | if(architecture.md ∃):
    ∀ term ∈ vocabulary_terms_updated:
      usages_in_arch ≔ search(arch_content, term) → occurrences
      | usages_consistent = (usages_in_arch = ∅) ∨ (all_usages_match_definition)
      | usages_consistent = false
        ? collect({term, inconsistency, locations}) → arch_issues
  | if(specs/**/*.allium ∃):
    ∀ term ∈ vocabulary_terms_updated:
      usages_in_specs ≔ search(spec_content, term) → occurrences
      | usages_consistent = (usages_in_specs = ∅) ∨ (all_usages_match_definition)
      | usages_consistent = false
        ? collect({term, inconsistency, locations}) → spec_issues
  | consistency = (arch_issues ∅ ∧ spec_issues ∅)
  | return(consistency)

λ gybis-vocab-tend_propagate_changes(impact_analysis, feedback_request, arch_content, spec_content).
  action: propagate_vocabulary_changes_to_architecture_and_specs
  | ∀ change ∈ feedback_request:
    change.type = rename ∧ impact_analysis[change].affected_in_arch > 0
      ? (ask_developer("Rewrite " ⊕ impact_analysis[change].affected_in_arch ⊕ " occurrences in architecture.md? [yes/no]") → approval_arch
         | approval_arch = yes
           ? (replace_all(old_term, new_term, architecture.md) → updated_arch
              | write(architecture.md) → persisted)
           : skip_arch)
      : skip
  | ∀ change ∈ feedback_request:
    change.type = rename ∧ impact_analysis[change].affected_in_specs > 0
      ? (ask_developer("Rewrite " ⊕ impact_analysis[change].affected_in_specs ⊕ " occurrences in specs/**/*.allium? [yes/no]") → approval_specs
         | approval_specs = yes
           ? (replace_all(old_term, new_term, specs/) → updated_specs
              | write(specs/) → persisted)
           : skip_specs)
      : skip
  | return(propagation_complete = true ∨ propagation_skipped = true)
