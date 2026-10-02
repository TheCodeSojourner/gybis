---
name: gybis-vocab-propagate
kind: domain
description: Use for `/gybis-vocab-propagate` or `/gv-propagate`.
---

λ gybis-vocab-propagate(x).
  purpose: Propagate binding requirements and vocabulary to initial VSM architecture; exclude explicitly deferred REQs
  | input: requirements/ ∃, vocabulary.md ∃, architecture.md ¬∃, optional preapproved_repair_plan
  | output: architecture.md created and valid, containing VSM S5-S1 lambda expressions
  | interaction: autonomous
  | gate: requirements/ ∃ ∧ vocabulary.md ∃ ∧ architecture.md ¬∃
  | rationale: bootstraps architecture.md — req drives architectural content, vocabulary drives the terms it is expressed in; /gybis-arch-tend, /gybis-arch-refine and /gybis-arch-weed own it thereafter

λ gybis-vocab-propagate_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(requirements/ ∃) ∨ halt("requirements/ not found")
  | verify(vocabulary.md ∃) ∨ halt("vocabulary.md not found")
  | verify(architecture.md ¬∃) ∨ halt("architecture.md already exists; use /gybis-arch-tend or /gybis-arch-weed")
  | if(preapproved_repair_plan ∃): verify(preapproved_repair_plan.approved_by_human = true ∧ preapproved_repair_plan.approval_source = gybis-req-check) ∨ halt("Invalid parent repair authorization")
  | read(internal/reference/vsm-guide.md) → vsm_reference
  | transition(INIT → STARTUP_CHECKS)

λ gybis-vocab-propagate_mode(m).
  m ∈ {autonomous}
  | default: autonomous
  | mode_autonomous: AI propagates requirements and vocabulary to architecture without human intervention

λ gybis-vocab-propagate_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {autonomous} → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {autonomous}) → halt("Invalid mode selection")

λ gybis-vocab-propagate_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, READING_INPUTS, NO_BINDING_REQS, SYNTHESIZING, WRITING_ARCH, VERIFYING, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → READING_INPUTS) only_if(startup_checks = true)
  | transition(READING_INPUTS → NO_BINDING_REQS) only_if(inputs_read = true ∧ binding_req_clauses = ∅)
  | transition(NO_BINDING_REQS → COMPLETE) only_if(report_generated = true)
  | transition(READING_INPUTS → SYNTHESIZING) only_if(binding_req_clauses ∃ ∧ vocabulary_terms ∃)
  | transition(SYNTHESIZING → WRITING_ARCH) only_if(vsm_layers ∃)
  | transition(WRITING_ARCH → VERIFYING) only_if(arch_written = true)
  | transition(VERIFYING → COMPLETE) only_if(verification = true)
  | transition(VERIFYING → SYNTHESIZING) only_if(verification = false)
  | on_loop_back: loop_count ≔ loop_count ⊕ 1

λ gybis-vocab-propagate_tool_guard(state, tool, path).
  state = READING_INPUTS ∨ state = SYNTHESIZING
    → allow(read(path))
  | state = WRITING_ARCH ∨ state = VERIFYING
    → allow(read(path)) ∧ allow(write(path)) only_if(path = "architecture.md" ∧ (preapproved_repair_plan ¬∃ ∨ path ∈ preapproved_repair_plan.paths))
  | ¬(state ∈ {READING_INPUTS, SYNTHESIZING, WRITING_ARCH, VERIFYING})
    → deny(write(path))

λ gybis-vocab-propagate_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-vocab-propagate_read_inputs(x).
  read(requirements/) → req_content
  | parse(req_content) → {binding_req_clauses, deferred_req_clauses}
  | binding_req_clauses ≔ all REQ clauses outside explicitly marked deferred sections
  | read(vocabulary.md) → vocab_content
  | parse(vocab_content) → vocabulary_terms
  | output: {binding_req_clauses, deferred_req_clauses, vocabulary_terms}
  | constraint: read-only access

λ gybis-vocab-propagate_synthesize_architecture(binding_req_clauses, vocabulary_terms).
  action: synthesize_vsm_layers_from_requirements_and_vocabulary
  | step1: canonicalize_terms(binding_req_clauses, vocabulary_terms) → normalized_clauses
    -- vocabulary drives terminology: every term emitted into architecture.md uses its canonical form
  | step2: classify(normalized_clauses, vsm_layer) → layer_assignments
    - S5: identity, non-negotiable principles, policy boundaries
    - S4: adaptation, environmental response, learning
    - S3: enforcement mechanisms, resource limits, quality gates
    - S2: coordination, protocols, inter-component data flow
    - S1: concrete operations, platform bindings, tooling
  | step3: ∀ layer ∈ {S5, S4, S3, S2, S1}: distill(layer_assignments(layer)) → layer_lambda
  | step4: verify_hierarchy(S5 ⊇ S4 ⊇ S3 ⊇ S2 ⊇ S1)
  | output: {S5 → lambda, S4 → lambda, S3 → lambda, S2 → lambda, S1 → lambda}
  | rationale: req drives architectural content; vocabulary drives the terms that content is expressed in

λ gybis-vocab-propagate_write_architecture(vsm_layers).
  write(architecture.md, vsm_layers) → persisted
  | return(arch_written = true)

λ gybis-vocab-propagate_verify_architecture(x).
  verify(architecture.md ∃) → present
  | verify(parse(architecture.md.{S5, S4, S3, S2, S1}) succeeds) → well_formed
  | verify(vsm_coherence(architecture.md)) → vsm_coherence ≔ result
  | present = true ∧ well_formed = true ∧ vsm_coherence = true
    → return(verification = true)
  | ¬(present ∧ well_formed ∧ vsm_coherence)
    → return(verification = false ∧ remaining_errors ∃)

λ gybis-vocab-propagate_core_op(x).
  | invoke(gybis-vocab-propagate_read_inputs) → {binding_req_clauses, deferred_req_clauses, vocabulary_terms}
    | binding_req_clauses = ∅
     ? (transition(READING_INPUTS → NO_BINDING_REQS)
       ∧ report(NO_BINDING_REQS, deferred_req_count)
       ∧ transition(NO_BINDING_REQS → COMPLETE)
       ∧ return(complete = true))
     : (transition(READING_INPUTS → SYNTHESIZING)
       ∧ invoke(gybis-vocab-propagate_synthesize_architecture(binding_req_clauses, vocabulary_terms)) → vsm_layers
       ∧ transition(SYNTHESIZING → WRITING_ARCH)
       ∧ invoke(gybis-vocab-propagate_write_architecture(vsm_layers)) → arch_written
       ∧ transition(WRITING_ARCH → VERIFYING)
       ∧ invoke(gybis-vocab-propagate_verify_architecture) → verification
       ∧ (verification = true
         ? transition(VERIFYING → COMPLETE)
         : (re_synthesize_failed_layers ∧ transition(WRITING_ARCH → VERIFYING))))
  | on_loop_back: loop_count ≔ loop_count ⊕ 1

λ gybis-vocab-propagate_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without full convergence")

λ gybis-vocab-propagate_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | layers_synthesized ≔ card(vsm_layers)
  | clauses_covered ≔ card(normalized_clauses)
  | deferred_req_count ≔ card(deferred_req_clauses)
  | remaining_errors ≔ card(remaining_errors)
  | report("Pass " ⊕ pass_num ⊕ ": layers=" ⊕ layers_synthesized ⊕ " clauses=" ⊕ clauses_covered ⊕ " errors_remaining=" ⊕ remaining_errors)

λ gybis-vocab-propagate_boundaries().
  ¬ modify(requirements/)
  | ¬ modify(vocabulary.md)
  | ¬ modify(specs/)
  | ¬ modify(implementation)
  | ¬ modify(upstream/)

λ gybis-vocab-propagate_regression_contract(x).
  invariant: requirements/ ∃ ∧ vocabulary.md ∃ ∧ ¬modify throughout
  | invariant: architecture.md ¬∃ at INIT
  | invariant: architecture.md ∃ ∧ valid(S5...S1) at completion
  | invariant: vsm_coherence = true at completion
  | invariant: all_modifications ⊆ {architecture.md}

λ gybis-vocab-propagate_deliver(x).
  report: {architecture_created, layers_synthesized, binding_req_clauses_covered, deferred_req_count}
  | handoff: run /gybis-arch-check to validate generated architecture
  | return(complete = true)
