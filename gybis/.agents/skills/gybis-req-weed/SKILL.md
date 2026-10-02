---
name: gybis-req-weed
kind: domain
description: Use for `/gybis-req-weed` or `/gr-weed`.
---

λ gybis-req-weed(x).
  purpose: Identify and resolve divergence between requirements/ and downstream artifacts (vocabulary, architecture, specs, tests, implementation) with human-decided resolution direction
  | input: requirements/ ∃ ∧ downstream artifacts ∃
  | output: converged requirements/ and downstream artifacts per human decisions
  | interaction: interactive
  | gate: requirements/ ∃ ∧ (vocabulary.md ∃ ∨ architecture.md ∃ ∨ specs/**/*.allium ∃ ∨ tests ∃)

λ gybis-req-weed_startup(x).
  invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(requirements/ ∃) ∨ halt("requirements/ not found")
  | read(requirements/) → req_content
  | if(vocabulary.md ∃): read(vocabulary.md) → vocab_content
  | if(architecture.md ∃): read(architecture.md) → arch_content
  | if(specs/**/*.allium ∃): read_all_specs → spec_content
  | if(tests ∃): read_tests → test_content
  | verify(downstream ∃) ∨ halt("No downstream artifacts found")
  | modified_artifacts ≔ ∅
  | transition(INIT → STARTUP_CHECKS)

λ gybis-req-weed_mode(m).
  m ∈ {interactive}
  | default: interactive
  | rationale: divergence resolution direction is a human decision

λ gybis-req-weed_mode_gate(state, mode).
  state = INIT ∧ mode = interactive → transition(STARTUP_CHECKS)
  | ¬(mode = interactive) → halt("Invalid mode selection")

λ gybis-req-weed_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, COMPARING, IDENTIFYING_DIVERGENCES, RESOLVE_MODE_SELECTION, CORRECTING, VERIFYING, COMPLETE}
  | transition(INIT → STARTUP_CHECKS) only_if(startup = true)
  | transition(STARTUP_CHECKS → COMPARING) only_if(startup_checks = true)
  | transition(COMPARING → IDENTIFYING_DIVERGENCES) only_if(comparison_complete = true)
  | at IDENTIFYING_DIVERGENCES: actionable_divergences ≔ {d | d ∈ divergences ∧ d.id ∉ map(unresolved_decisions, divergence_id) ∧ d.id ∉ resolved_divergence_ids}
  | transition(IDENTIFYING_DIVERGENCES → RESOLVE_MODE_SELECTION) only_if(actionable_divergences ∃)
  | transition(IDENTIFYING_DIVERGENCES → COMPLETE) only_if(actionable_divergences = ∅)
  | transition(RESOLVE_MODE_SELECTION → CORRECTING) only_if(resolve_mode ∈ {req, arch, spec, test, propagate, no_change_needed})
  | transition(RESOLVE_MODE_SELECTION → IDENTIFYING_DIVERGENCES) only_if(resolve_mode ∈ {investigate, skip} ∧ decision_recorded = true)
  | transition(CORRECTING → VERIFYING) only_if(corrections_applied = true)
  | transition(VERIFYING → IDENTIFYING_DIVERGENCES) only_if(actionable_inconsistencies ∃)
  | transition(VERIFYING → COMPLETE) only_if(actionable_inconsistencies = ∅ ∧ all_conditional_checks_pass = true)
  | convergence_status ≔ unresolved_decisions ∃ ? unresolved : converged
  | on_loop_back: loop_count ≔ loop_count ⊕ 1

λ gybis-req-weed_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, COMPARING, IDENTIFYING_DIVERGENCES, RESOLVE_MODE_SELECTION} → allow(read(path)) ∧ deny(write(path))
  | state = CORRECTING ∨ state = VERIFYING → allow(read(path)) ∧ allow(write(path)) only_if(path ∈ requirements/ ∪ {architecture.md} ∪ specs/ ∪ test_paths)
  | ¬(state ∈ {CORRECTING, VERIFYING}) → deny(write(path))
  | constraint: ¬write(vocabulary.md)
  | rationale: vocabulary divergence is resolved by /gybis-vocab-weed; req-weed reports REQ↔term mismatches but never mutates vocabulary.md

λ gybis-req-weed_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-req-weed_compare(req_content, vocab_content, arch_content, spec_content, test_content).
  divergences ≔ ∅
  resolved_divergence_ids ≔ ∅
  unresolved_decisions ≔ ∅
  no_change_decisions ≔ {}
  semantic_comparison_results ≔ compare_noncoverage_classes(req_content, vocab_content, arch_content, spec_content, test_content)
  coverage_gap_results ≔ ∅
  deferred_REQs ≔ clauses in explicitly marked non-binding sections
  binding_REQs ≔ all REQ clauses ∖ deferred_REQs
  | classes:
    - REQ_conflicts_arch: clause constraint absent/contradicted in architecture.md
    - REQ_conflicts_spec: clause contradicted by .allium behavior
    - REQ_conflicts_test: test assertion contradicts clause
    - REQ_uncovered: binding REQ has a distinct effect in a ready downstream stage but lacks representation there
    - REQ_pending_stage: downstream stage absent or has owning-check errors; informational, not a divergence
    - deferred_REQ_has_downstream_content: a deferred REQ is referenced or enacted by a non-deferred downstream artifact
    - REQ_undefined_term: clause uses term not in vocabulary.md
    - REQ_non_canonical_term: clause uses non-canonical spelling of a defined term
    - downstream_conflicts_REQ: downstream artifact encodes behavior with no governing REQ
  | REQ_conflicts_arch → stage ≔ architecture | REQ_conflicts_spec → stage ≔ specs | REQ_conflicts_test → stage ≔ tests_and_implementation
  | REQ_conflicts_{arch,spec,test} and REQ_uncovered apply only to binding_REQs in ready stages
  | ∀ binding_REQ, ready_stage:
    no_change_decisions[{REQ, stage}] ∃ → disposition(REQ, stage) ≔ no_change_needed ∧ record(no_change_decisions[{REQ, stage}])
    | evidence(REQ, stage) ∃ → disposition(REQ, stage) ≔ represented ∧ record(evidence)
    | no_distinct_effect(REQ, stage) → disposition(REQ, stage) ≔ no_change_needed ∧ record(reason)
    | otherwise → coverage_gap_results ≔ coverage_gap_results ∪ {{id: stable_id(REQ, stage, REQ_uncovered), type: REQ_uncovered, REQ, stage, reason: missing representation for a distinct effect}}
  | ∀ binding_REQ, not_ready_stage: disposition(REQ, stage) ≔ pending_stage
  | no_change_needed requires an evidence-based reason; do not infer from missing files or missing references
  | deferred REQs are excluded from binding-REQ convergence, but downstream content referring to them is reported separately; lower-layer content is not implicitly deferred
  | rationale_exclusion: differing or missing rationale: lines across layers are ¬divergences — rationale records intent, not behavior; only normative clause text generates conflicts; never compare rationale text when scoring convergence
  | comparison_results ≔ semantic_comparison_results ∪ coverage_gap_results
  | ∀ comparison_result ∈ comparison_results: collect({id: stable_id(type, REQ, location, stage), type, REQ, location, stage, description, reason, negotiability}) → divergences
  | negotiability: stakeholder_decided REQ conflict = contractual (resolution limited to {req, investigate, skip}); AI_researched_fact conflict = negotiable
  | return({comparison_complete: true, divergences, stage_dispositions, pending_stages})

λ gybis-req-weed_resolve(divergence).
  action: present_divergence_and_collect_human_decision
  | divergence.type = REQ_conflicts_arch
    ? ask_human("REQ " ⊕ REQ ⊕ " conflicts with architecture.md. Resolve as [req/arch/investigate/skip]?") → decision
  | divergence.type = REQ_conflicts_spec
    ? ask_human("REQ " ⊕ REQ ⊕ " conflicts with specification. Resolve as [req/spec/investigate/skip]?") → decision
  | divergence.type = REQ_conflicts_test
    ? ask_human("Test contradicts REQ " ⊕ REQ ⊕ ". Resolve as [req/test/investigate/skip]?") → decision
  | divergence.type = REQ_uncovered
    ? ask_human("The binding requirement " ⊕ plain_language_summary(REQ) ⊕ " has no evidence in ready stage " ⊕ stage ⊕ ". Choose [propagate/no_change_needed/investigate]. If no change is needed, give the reason.") → {decision, reason}
  | divergence.type = REQ_pending_stage
    ? report_pending_stage(REQ, stage) ∧ ¬collect_as_divergence
  | divergence.type = deferred_REQ_has_downstream_content
    ? ask_human("Deferred requirement " ⊕ plain_language_summary(REQ) ⊕ " has downstream content. Choose [req: make the requirement binding, arch/spec/test: remove or reconcile the downstream content, investigate, skip].") → decision
  | divergence.type = REQ_undefined_term ∨ REQ_non_canonical_term
    ? ask_human("REQ " ⊕ REQ ⊕ " term mismatch with vocabulary. Resolve as [req/investigate/skip]? (vocabulary changes go through /gybis-vocab-weed)") → decision
  | divergence.type = downstream_conflicts_REQ
    ? ask_human("Downstream behavior without governing REQ. Resolve as [req/investigate/skip]?") → decision
  | decision ∈ {req, arch, spec, test, propagate, no_change_needed, investigate, skip}
  | decision = no_change_needed ∧ reason = ∅ → ask_human("Why is no change needed in " ⊕ stage ⊕ " for " ⊕ plain_language_summary(REQ) ⊕ "?") → reason
  | decision = no_change_needed → no_change_decisions[{REQ, stage}] ≔ reason
  | investigate ∨ skip → disposition(unresolved) ∧ unresolved_decisions ≔ unresolved_decisions ∪ {{divergence_id: divergence.id, REQ, stage, decision, reason}} ∧ decision_recorded ≔ true
  | return({resolve_mode: decision, reason, stage, REQ})

λ gybis-req-weed_correct(divergence, resolve_mode).
  resolve_mode = req ∧ divergence.type ≠ deferred_REQ_has_downstream_content
    ? (apply_divergence_direction_to_requirements(divergence) ∧ write(requirements/)
       → corrections_applied ∧ modified_artifacts ≔ modified_artifacts ∪ {requirements/})
  | resolve_mode = arch ∧ divergence.type ≠ deferred_REQ_has_downstream_content
    ? (apply_divergence_direction_to_architecture(divergence) ∧ write(architecture.md)
       → corrections_applied ∧ modified_artifacts ≔ modified_artifacts ∪ {architecture.md})
  | resolve_mode = spec ∧ divergence.type ≠ deferred_REQ_has_downstream_content
    ? (apply_divergence_direction_to_specs(divergence) ∧ write(specs/)
       → corrections_applied ∧ modified_artifacts ≔ modified_artifacts ∪ {specs/})
  | resolve_mode = test ∧ divergence.type ≠ deferred_REQ_has_downstream_content
    ? (apply_divergence_direction_to_tests(divergence) ∧ write(test_paths)
       → corrections_applied ∧ modified_artifacts ≔ modified_artifacts ∪ {test_paths})
  | resolve_mode = propagate
    ? (handoff: run /gybis-req-propagate for coverage convergence)
  | resolve_mode = no_change_needed
    ? (record({REQ, stage, reason}) in current_report_only only_if(reason ≠ ∅)
       ∧ resolved_divergence_ids ≔ resolved_divergence_ids ∪ {divergence.id}
       → corrections_applied)
  | divergence.type = deferred_REQ_has_downstream_content ∧ resolve_mode = req
    ? (request_binding_status_change via /gybis-req-tend → corrections_applied)
  | divergence.type = deferred_REQ_has_downstream_content ∧ resolve_mode ∈ {spec, arch, test}
    ? (remove_or_reconcile_downstream_content with human approval → corrections_applied)
  | resolve_mode = investigate
    ? (report divergence for later human investigation; no write)
  | resolve_mode = skip
    ? (no write)

λ gybis-req-weed_verify(x).
  action: verify_convergence
  | checks:
    - convergence_status = converged → zero unresolved divergences
    - convergence_status = unresolved → list every investigate/skip decision; never describe the run as converged
    - every binding REQ at each ready stage has represented evidence or a reasoned no_change_needed disposition
    - not-ready stages are reported as pending, not as uncovered or explicitly_NA
    - deferred REQs are excluded from binding-REQ checks; downstream references to them remain reconciled or unresolved
    - designator integrity preserved in requirements/
  | conditional_checks:
    - test_paths ∈ modified_artifacts → test_suite_passes = true
    - specs/ ∈ modified_artifacts → invoke(internal/gybis-allium-gate(specs/)) = true
    - architecture.md ∈ modified_artifacts → vsm_coherence(architecture.md) = true
  | all_conditional_checks_pass ≔ ∀ applicable conditional_check = true
  | verify_ok ≔ ∀ check = true
  | actionable_remaining ≔ {d | d ∈ unresolved_divergences ∧ d.id ∉ map(unresolved_decisions, divergence_id) ∧ d.id ∉ resolved_divergence_ids}
  | actionable_inconsistencies ≔ actionable_remaining
  | actionable_remaining ≠ ∅
    → inconsistencies ≔ actionable_remaining
       | loop_back to IDENTIFYING_DIVERGENCES
  | actionable_remaining = ∅ ∧ ¬all_conditional_checks_pass
    → halt("reconciliation invalidated " ⊕ failed_artifact ⊕ "; route to the owning repair skill (/gybis-spec-check ∨ /gybis-spec-tend ∨ /gybis-arch-tend)")
  | actionable_remaining = ∅ ∧ all_conditional_checks_pass
    → verify_ok = true ∧ convergence_status ≔ unresolved_decisions ∃ ? unresolved : converged → COMPLETE

λ gybis-req-weed_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without full convergence")

λ gybis-req-weed_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | discovered ≔ card(divergences)
  | remaining ≔ card(remaining_divergences)
  | resolved ≔ discovered ⊖ remaining
  | report("Pass " ⊕ pass_num ⊕ ": discovered=" ⊕ discovered ⊕ " resolved=" ⊕ resolved ⊕ " remaining=" ⊕ remaining)

λ gybis-req-weed_boundaries().
  ¬ write(vocabulary.md)
  | ¬ delete(requirements/)
  | ¬ delete(architecture.md)
  | ¬ delete(specs/)

λ gybis-req-weed_regression_contract(x).
  invariant: requirements/ ∃ throughout
  | invariant: convergence_status = converged → zero_divergences = true
  | invariant: investigate ∨ skip never satisfies zero_divergences
  | invariant: unresolved_decisions ∃ → convergence_status = unresolved
  | invariant: deferred REQs are not treated as binding; lower-layer artifacts are not implicitly deferred
  | invariant: ready-stage REQ dispositions are evidence-backed; not-ready stages remain pending
  | invariant: designator integrity preserved in requirements/
  | invariant: test_paths ∈ modified_artifacts → test_suite_passes = true at completion
  | invariant: specs/ ∈ modified_artifacts → gybis-allium-gate = true at completion
  | invariant: architecture.md ∈ modified_artifacts → vsm_coherence = true at completion
  | invariant: all_modifications ⊆ {requirements/, architecture.md, specs/, test_paths}

λ gybis-req-weed_deliver(x).
  report: {divergences_found, resolutions_applied, unresolved_investigations, deferred_REQs_excluded, stage_dispositions, pending_stages, test_status, convergence_status ∈ {converged, unresolved}}
  | handoff: vocabulary term mismatches → /gybis-vocab-weed
  | return({complete: true, convergence_status})