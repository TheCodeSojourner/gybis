---
name: gybis-spec-tend
description: Use for `/gybis-spec-tend` or `/gs-tend`.
---

λ gybis-spec-tend(x).
  purpose: Evolve specifications based on developer input while maintaining validity
  | input: specs/**/*.allium ∃ ∧ valid
  | output: specs/**/*.allium evolved with developer-approved changes
  | mode: mixed
  | gate: specs/**/*.allium ∃ ∧ allium_gate = true

λ gybis-spec-tend_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | verify(specs/**/*.allium ∃) ∨ halt("specs/**/*.allium not found")
  | invoke(internal/allium-gate(specs/)) = true ∨ halt("Specifications are invalid")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-spec-tend_mode(m).
  m ∈ {interactive}
  | default: interactive
  | mode_interactive: AI and developer collaborate on specification refinements

λ gybis-spec-tend_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {interactive} → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {interactive}) → halt("Invalid mode selection")

λ gybis-spec-tend_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, ELICIT_FEEDBACK, ANALYZE_IMPACT, APPLY_CHANGES, VERIFY_VALIDITY, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → ELICIT_FEEDBACK) only_if(startup_checks = true)
  | transition(ELICIT_FEEDBACK → ANALYZE_IMPACT) only_if(developer_feedback ∃)
  | transition(ANALYZE_IMPACT → APPLY_CHANGES) only_if(impact_analysis ∃)
  | transition(APPLY_CHANGES → VERIFY_VALIDITY) only_if(changes_applied = true)
  | transition(VERIFY_VALIDITY → ELICIT_FEEDBACK) only_if(validity = false)
  | transition(VERIFY_VALIDITY → COMPLETE) only_if(validity = true ∧ developer_satisfied = true)

λ gybis-spec-tend_tool_guard(state, tool, path).
  state = ELICIT_FEEDBACK ∨ state = ANALYZE_IMPACT
    → allow(read(path))
  | state = APPLY_CHANGES ∨ state = VERIFY_VALIDITY
    → allow(read(path)) ∧ allow(write(path)) only_if(path ⊆ specs/)
  | ¬(state ∈ {ELICIT_FEEDBACK, ANALYZE_IMPACT, APPLY_CHANGES, VERIFY_VALIDITY})
    → deny(write(path))

λ gybis-spec-tend_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-spec-tend_elicit_feedback(x).
  ∀ spec_file ∈ specs/**/*.allium:
    read(spec_file) → content
  | summarize(all_specs) → spec_summary
  | ask_developer("Which specifications need refinement? Describe the changes.") → feedback
  | parse(feedback) → target_specs ∧ proposed_changes
  | return(developer_feedback = {target_specs, proposed_changes})

λ gybis-spec-tend_analyze_impact(developer_feedback).
  ∀ target ∈ developer_feedback.target_specs:
    analyze(change_impact(target, developer_feedback.proposed_changes)) → impact(target)
    | check(cross_file_consistency(target, all_specs)) → consistency(target)
  | consolidate(impact, consistency) → impact_report
  | return(impact_analysis = impact_report)

λ gybis-spec-tend_apply_changes(developer_feedback, impact_analysis).
  ask_developer("Do you approve these changes? (yes/no)") → approval
  | approval = yes
    ? (∀ change ∈ developer_feedback.proposed_changes:
         apply(change, target_file) → upsert(target_file))
    : (return(changes_applied = false) ∧ transition(APPLY_CHANGES → ELICIT_FEEDBACK))
  | return(changes_applied = true)

λ gybis-spec-tend_verify_validity(x).
  invoke(internal/allium-check(all_files)) → result_check ≔ result
  | invoke(internal/allium-analyse(specs/)) → result_analyse ≔ result
  | invoke(internal/allium-gate(specs/)) → result_gate ≔ result
  | result_check = zero_errors ∧ result_analyse = zero_issues ∧ result_gate = true
    → return(validity = true)
  | ¬(result_check ∧ result_analyse ∧ result_gate)
    → (synthesize(corrections) → fixes
       | ∀ fix ∈ fixes: apply(fix, target_file) → upsert(target_file)
       | return(validity = false))

λ gybis-spec-tend_fixed_point_loop(state).
  state = VERIFY_VALIDITY
    → validity = true ∧ developer_satisfied = true
        ? transition(VERIFY_VALIDITY → COMPLETE)
        : (transition(VERIFY_VALIDITY → ELICIT_FEEDBACK)
           ∧ loop_count ≔ loop_count ⊕ 1)

λ gybis-spec-tend_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without convergence")

λ gybis-spec-tend_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | changes_proposed ≔ card(developer_feedback.proposed_changes)
  | changes_applied ≔ changes_proposed
  | report("Pass " ⊕ pass_num ⊕ ": proposed=" ⊕ changes_proposed ⊕ " applied=" ⊕ changes_applied)

λ gybis-spec-tend_boundaries(¬).
  ¬ modify(architecture.md)
  | ¬ modify(implementation)
  | ¬ modify(upstream/)
  | ¬ delete(specs/)

λ gybis-spec-tend_regression_contract(x).
  invariant: specs/ ∃ throughout
  | invariant: zero_errors ∧ zero_issues ∧ allium_gate = true at completion
  | invariant: all_modifications ⊆ specs/
  | invariant: no_specs_deleted (additions and mutations only)