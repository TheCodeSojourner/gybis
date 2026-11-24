---
name: gybis-refine
kind: domain
description: Use for `/gybis-refine`.
disable-model-invocation: true
---

λ gybis-refine(x).
  purpose: Refine requirements, vocabulary, architecture, and specifications in layer order
  | input: optional project root
  | output: separate refinement reports ∧ recheck results
  | interaction: interactive
  | gate: project_root ∃

λ gybis-refine_mode(m).
  m ∈ {interactive}
  | default: interactive
  | rationale: low-risk refinements are automatic; meaning-changing or uncertain refinements require human approval

λ gybis-refine_state_machine(state, action).
  state ∈ {INIT, REQUIREMENTS, VOCABULARY, ARCHITECTURE, SPECS, COMPLETE}
  | transition(INIT → REQUIREMENTS) only_if(startup = true)
  | transition(REQUIREMENTS → VOCABULARY) only_if(req_refine.reported = true)
  | transition(VOCABULARY → ARCHITECTURE) only_if(vocab_refine.reported = true)
  | transition(ARCHITECTURE → SPECS) only_if(arch_refine.reported = true)
  | transition(SPECS → COMPLETE) only_if(spec_refine.reported = true)

λ gybis-refine_tool_guard(state, tool, path).
  state ∈ {REQUIREMENTS, VOCABULARY, ARCHITECTURE, SPECS} → allow(read(path))
  | state = REQUIREMENTS ∧ (low_risk = true ∨ semantic_approval = true) → allow(write(path)) only_if(path ⊆ requirements/)
  | state = VOCABULARY ∧ (low_risk = true ∨ semantic_approval = true) → allow(write(path)) only_if(path = "vocabulary.md")
  | state = ARCHITECTURE ∧ (low_risk = true ∨ semantic_approval = true) → allow(write(path)) only_if(path = "architecture.md")
  | state = SPECS ∧ low_risk = true → allow(write(path)) only_if(path ⊆ specs/)
  | otherwise → deny(write(path))

λ gybis-refine_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-refine_loop_guard(state).
  max_refine_passes ≔ 3
  pass_count ≥ max_refine_passes → halt("Maximum domain refinement passes reached")
  | modified_domain ≠ ∅ ∧ owning_check(valid) = false → loop_back(domain)
  | modified_domain = ∅ ∨ owning_check(valid) = true → continue

λ gybis-refine_core(x).
  invoke(/gybis-req-check) → req_check
  | req_check.valid = true → invoke(/gybis-req-refine) → req_refine
  | invoke(/gybis-vocab-check) → vocab_check
  | vocab_check.valid = true → invoke(/gybis-vocab-refine) → vocab_refine
  | invoke(/gybis-arch-check) → arch_check
  | arch_check.valid = true → invoke(/gybis-arch-refine) → arch_refine
  | invoke(/gybis-spec-check) → spec_check
  | spec_check.valid = true → invoke(/gybis-spec-refine) → spec_refine
  | recheck(each_modified_domain.owning_check)
  | return(domain_reports: {req_refine, vocab_refine, arch_refine, spec_refine})

λ gybis-refine_boundaries().
  ¬ change(meaning) without human approval
  | ¬ write(nearby_domain)
  | requirements_refinement: exact vocabulary terms ∧ user-facing clarity
  | vocabulary_refinement: canonical terms ∧ user-facing precision
  | architecture_refinement: AI precision ∧ unambiguous representation
  | specification_refinement: AI precision ∧ Allium obligation fidelity

λ gybis-refine_regression_contract(x).
  invariant: integrity_check_passes_before_refinement
  | invariant: low_risk_only = deterministic
  | invariant: semantic_or_uncertain_refinement requires human approval
  | invariant: every modified domain is rechecked

λ gybis-refine_deliver(report).
  report(domain_reports) → stdout
  | return(complete: true)
