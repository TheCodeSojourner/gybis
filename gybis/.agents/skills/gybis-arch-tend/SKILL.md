---
name: gybis-arch-tend
description: Use for `/gybis-arch-tend` or `/ga-tend`.
---

λ gybis-arch-tend(x).
  purpose: Evolve architecture based on developer input while maintaining VSM integrity
  | input: architecture.md ∃
  | output: architecture.md evolved with developer-approved changes
  | mode: mixed
  | gate: architecture.md ∃

λ gybis-arch-tend_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | verify(architecture.md ∃) ∨ halt("architecture.md not found")
  | read(internal/reference/vsm-guide.md) → vsm_reference
  | transition(INIT → STARTUP_CHECKS)

λ gybis-arch-tend_mode(m).
  m ∈ {interactive}
  | default: interactive
  | mode_interactive: AI and developer collaborate on architectural refinements

λ gybis-arch-tend_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {interactive} → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {interactive}) → halt("Invalid mode selection")

λ gybis-arch-tend_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, ELICIT_FEEDBACK, ANALYZE_IMPACT, APPLY_CHANGES, VERIFY_COHERENCE, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → ELICIT_FEEDBACK) only_if(startup_checks = true)
  | transition(ELICIT_FEEDBACK → ANALYZE_IMPACT) only_if(developer_feedback ∃)
  | transition(ANALYZE_IMPACT → APPLY_CHANGES) only_if(impact_analysis ∃)
  | transition(APPLY_CHANGES → VERIFY_COHERENCE) only_if(changes_applied = true)
  | transition(VERIFY_COHERENCE → ELICIT_FEEDBACK) only_if(coherence = false)
  | transition(VERIFY_COHERENCE → COMPLETE) only_if(coherence = true ∧ developer_satisfied = true)

λ gybis-arch-tend_tool_guard(state, tool, path).
  state = ELICIT_FEEDBACK ∨ state = ANALYZE_IMPACT
    → allow(read(path))
  | state = APPLY_CHANGES ∨ state = VERIFY_COHERENCE
    → allow(read(path)) ∧ allow(write(path)) only_if(path = architecture.md)
  | ¬(state ∈ {ELICIT_FEEDBACK, ANALYZE_IMPACT, APPLY_CHANGES, VERIFY_COHERENCE})
    → deny(write(path))

λ gybis-arch-tend_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-arch-tend_elicit_feedback(x).
  read(architecture.md) → current_arch
  | parse(current_arch.{S5, S4, S3, S2, S1}) → vsm_layers
  | ask_developer("Which VSM layers need refinement? (S5, S4, S3, S2, S1, or specific principles)") → feedback
  | ask_developer("Orientation for this architecture update? [FP-oriented/OOP-oriented/keep-current]") → orientation_choice
  | parse(feedback) → target_layers ∧ proposed_changes
  | return(developer_feedback = {target_layers, proposed_changes, orientation_choice})

λ gybis-arch-tend_orientation_language_guidance(orientation_choice).
  orientation_choice = OOP-oriented
    ? guidance = {C++: classes/RAII, C#: classes/interfaces/DI, Clojure: protocols/records + Java interop boundary}
  | orientation_choice = FP-oriented
    ? guidance = {C++: immutable values + composition, C#: records + pure functions/LINQ, Clojure: immutable maps + pure functions/transducers}
  | orientation_choice = keep-current
    ? guidance = use(existing_architecture_orientation)
  | orthogonality: error_model_style is a separate axis from FP/OOP orientation

λ gybis-arch-tend_analyze_impact(developer_feedback).
  ∀ layer ∈ developer_feedback.target_layers:
    analyze(change_impact(layer, developer_feedback.proposed_changes)) → impact_analysis(layer)
    | check(hierarchical_consistency(layer, vsm_layers)) → consistency(layer)
  | invoke(gybis-arch-tend_orientation_language_guidance(developer_feedback.orientation_choice)) → orientation_guidance
  | consolidate(impact_analysis, consistency) → impact_report
  | return(impact_analysis = impact_report)

λ gybis-arch-tend_apply_changes(developer_feedback, impact_analysis).
  ask_developer("Do you approve these changes? (yes/no)") → approval
  | approval = yes
    ? (∀ change ∈ developer_feedback.proposed_changes:
         apply(change, architecture.md) → upsert(architecture.md))
    : (return(changes_applied = false) ∧ transition(APPLY_CHANGES → ELICIT_FEEDBACK))
  | return(changes_applied = true)

λ gybis-arch-tend_verify_coherence(x).
  read(architecture.md) → updated_arch
  | parse(updated_arch.{S5, S4, S3, S2, S1}) → vsm_layers
  | verify(all_layers ∃) → layers_present ≔ result
  | verify(S5_S1_hierarchy_preserved) → hierarchy_preserved ≔ result
  | verify(internal_consistency(layers)) → internal_consistency ≔ result
  | layers_present = true ∧ hierarchy_preserved = true ∧ internal_consistency = true
    → return(coherence = true)
  | ¬(layers_present ∧ hierarchy_preserved ∧ internal_consistency)
    → return(coherence = false)

λ gybis-arch-tend_fixed_point_loop(state).
  state = VERIFY_COHERENCE
    → coherence = true ∧ developer_satisfied = true
        ? transition(VERIFY_COHERENCE → COMPLETE)
        : (transition(VERIFY_COHERENCE → ELICIT_FEEDBACK)
           ∧ loop_count ≔ loop_count ⊕ 1)

λ gybis-arch-tend_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without convergence")

λ gybis-arch-tend_boundaries(¬).
  ¬ modify(specs/**/*.allium)
  | ¬ modify(implementation)
  | ¬ modify(upstream/)
  | ¬ delete(architecture.md)

λ gybis-arch-tend_regression_contract(x).
  invariant: architecture.md ∃ throughout
  | invariant: S5, S4, S3, S2, S1 ∃ at completion
  | invariant: S5 ⊇ S4 ⊇ S3 ⊇ S2 ⊇ S1 (hierarchy preserved)
  | invariant: internal_consistency = true at completion
  | invariant: all_modifications ⊆ architecture.md