---
name: gybis-req-weed
description: Use for `/gybis-req-weed` or `/gr-weed`.
---

λ gybis-req-weed(x).
  purpose: Identify and resolve divergence between requirements/ and downstream artifacts (vocabulary, architecture, specs, tests, implementation) with human-decided resolution direction
  | input: requirements/ ∃ ∧ downstream artifacts ∃
  | output: converged requirements/ and downstream artifacts per human decisions
  | mode: interactive
  | gate: requirements/ ∃ ∧ (vocabulary.md ∃ ∨ architecture.md ∃ ∨ specs/**/*.allium ∃ ∨ tests ∃)

λ gybis-req-weed_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(requirements/ ∃) ∨ halt("requirements/ not found")
  | read(requirements/) → req_content
  | if(vocabulary.md ∃): read(vocabulary.md) → vocab_content
  | if(architecture.md ∃): read(architecture.md) → arch_content
  | if(specs/**/*.allium ∃): read_all_specs → spec_content
  | if(tests ∃): read_tests → test_content
  | verify(downstream ∃) ∨ halt("No downstream artifacts found")
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
  | transition(IDENTIFYING_DIVERGENCES → RESOLVE_MODE_SELECTION) only_if(divergences ∃)
  | transition(IDENTIFYING_DIVERGENCES → COMPLETE) only_if(divergences = ∅)
  | transition(RESOLVE_MODE_SELECTION → CORRECTING) only_if(resolve_mode ∃)
  | transition(CORRECTING → VERIFYING) only_if(corrections_applied = true)
  | transition(VERIFYING → IDENTIFYING_DIVERGENCES) only_if(inconsistencies ∃)
  | transition(VERIFYING → COMPLETE) only_if(consistency = true ∧ zero_divergences = true)

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
  | classes:
    - REQ_conflicts_arch: clause constraint absent/contradicted in architecture.md
    - REQ_conflicts_spec: clause contradicted by .allium behavior
    - REQ_conflicts_test: test assertion contradicts clause
    - REQ_uncovered: no spec/test target and no explicit_NA
    - REQ_undefined_term: clause uses term not in vocabulary.md
    - REQ_non_canonical_term: clause uses non-canonical spelling of a defined term
    - downstream_conflicts_REQ: downstream artifact encodes behavior with no governing REQ
  | rationale_exclusion: differing or missing rationale: lines across layers are ¬divergences — rationale records intent, not behavior; only normative clause text generates conflicts; never compare rationale text when scoring convergence
  | ∀ comparison result: collect({type, REQ, location, description, negotiability}) → divergences
  | negotiability: stakeholder_decided REQ conflict = contractual (resolution limited to {req, investigate, skip}); AI_researched_fact conflict = negotiable
  | return(comparison_complete = true ∧ divergences)

λ gybis-req-weed_resolve(divergence).
  action: present_divergence_and_collect_human_decision
  | divergence.type = REQ_conflicts_arch
    ? ask_human("REQ " ⊕ REQ ⊕ " conflicts with architecture.md. Resolve as [req/arch/investigate/skip]?") → decision
  | divergence.type = REQ_conflicts_spec
    ? ask_human("REQ " ⊕ REQ ⊕ " conflicts with specification. Resolve as [req/spec/investigate/skip]?") → decision
  | divergence.type = REQ_conflicts_test
    ? ask_human("Test contradicts REQ " ⊕ REQ ⊕ ". Resolve as [req/test/investigate/skip]?") → decision
  | divergence.type = REQ_uncovered
    ? ask_human("REQ " ⊕ REQ ⊕ " has no spec/test coverage. Resolve as [propagate/na/investigate/skip]?") → decision
  | divergence.type = REQ_undefined_term ∨ REQ_non_canonical_term
    ? ask_human("REQ " ⊕ REQ ⊕ " term mismatch with vocabulary. Resolve as [req/investigate/skip]? (vocabulary changes go through /gybis-vocab-weed)") → decision
  | divergence.type = downstream_conflicts_REQ
    ? ask_human("Downstream behavior without governing REQ. Resolve as [req/investigate/skip]?") → decision
  | decision ∈ {req, arch, spec, test, propagate, na, investigate, skip}
  | return(resolve_mode = decision)

λ gybis-req-weed_correct(divergence, resolve_mode).
  resolve_mode = req
    ? (apply_divergence_direction_to_requirements(divergence) ∧ write(requirements/) → corrections_applied)
  | resolve_mode = arch
    ? (apply_divergence_direction_to_architecture(divergence) ∧ write(architecture.md) → corrections_applied)
  | resolve_mode = spec
    ? (apply_divergence_direction_to_specs(divergence) ∧ write(specs/) → corrections_applied)
  | resolve_mode = test
    ? (apply_divergence_direction_to_tests(divergence) ∧ write(test_paths) → corrections_applied)
  | resolve_mode = propagate
    ? (handoff: run /gybis-req-propagate for coverage convergence)
  | resolve_mode = na
    ? (record explicit_NA with rationale in clause attribution → corrections_applied)
  | resolve_mode = investigate
    ? (report divergence for later human investigation; no write)
  | resolve_mode = skip
    ? (no write)

λ gybis-req-weed_verify(x).
  action: verify_convergence
  | checks:
    - zero unresolved divergences (or explicitly deferred via investigate/skip)
    - designator integrity preserved in requirements/
    - test_suite_passes = true (strict convergence when tests were modified)
  | verify_ok ≔ ∀ check = true
  | on fail: loop_back to IDENTIFYING_DIVERGENCES

λ gybis-req-weed_deliver(x).
  report: {divergences_found, resolutions_applied, deferred, test_status}
  | handoff: vocabulary term mismatches → /gybis-vocab-weed
  | return(complete = true)