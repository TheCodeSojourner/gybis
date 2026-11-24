---
name: gybis-weed
kind: domain
description: Use for `/gybis-weed`.
disable-model-invocation: true
---

λ gybis-weed(x).
  purpose: Resolve divergence across adjacent requirements/vocabulary, vocabulary/architecture, architecture/specs, and specs/tests/code boundaries
  | input: optional project root
  | output: adjacent_boundary_reports ∧ unresolved_human_decisions
  | interaction: interactive
  | gate: project_root ∃

λ gybis-weed_mode(m).
  m ∈ {interactive}
  | default: interactive
  | rationale: crossing a domain boundary requires ownership and human decisions for unresolved or semantic divergence

λ gybis-weed_state_machine(state, action).
  state ∈ {INIT, REQ_VOCAB, VOCAB_ARCH, ARCH_SPECS, SPECS_TESTS_CODE, COMPLETE}
  | transition(INIT → REQ_VOCAB) only_if(startup = true)
  | transition(REQ_VOCAB → VOCAB_ARCH) only_if(req_vocab.reported = true)
  | transition(VOCAB_ARCH → ARCH_SPECS) only_if(vocab_arch.reported = true)
  | transition(ARCH_SPECS → SPECS_TESTS_CODE) only_if(arch_specs.reported = true)
  | transition(SPECS_TESTS_CODE → COMPLETE) only_if(spec_test_code.reported = true)

λ gybis-weed_tool_guard(state, tool, path).
  state = REQ_VOCAB → allow(read(path)) ∧ allow(write(path)) only_if(human_approved = true ∧ path ∈ {requirements/, vocabulary.md})
  | state = VOCAB_ARCH → allow(read(path)) ∧ allow(write(path)) only_if(human_approved = true ∧ path ∈ {vocabulary.md, architecture.md})
  | state = ARCH_SPECS → allow(read(path)) ∧ allow(write(path)) only_if(human_approved = true ∧ path ∈ {architecture.md, specs/**/*.allium})
  | state = SPECS_TESTS_CODE → allow(read(path)) ∧ allow(write(path)) only_if(human_approved = true ∧ path ∈ {specs/**/*.allium, tests/, implementation})
  | otherwise → deny(write(path))

λ gybis-weed_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state")

λ gybis-weed_loop_guard(state).
  max_weed_passes ≔ 3
  pass_count ≥ max_weed_passes → preserve(unresolved_divergences) ∧ halt("Maximum adjacent-boundary weed passes reached")
  | modified_artifact ∃ → invoke(modified_artifact.owning_check_or_verifier)
  | verification_passes = true → continue
  | verification_passes = false → loop_back(adjoining_owner)

λ gybis-weed_core(x).
  invoke(/gybis-req-weed) → req_vocab
  | req_vocab.pending = true → preserve(unresolved_divergences)
  | invoke(/gybis-vocab-weed) → vocab_arch
  | invoke(/gybis-arch-weed) → arch_specs
  | invoke(/gybis-spec-weed) → spec_test_code
  | recheck(each_modified_artifact.owning_check_or_verifier)
  | return(boundary_reports: {req_vocab, vocab_arch, arch_specs, spec_test_code})

λ gybis-weed_boundaries().
  req_vocab: only requirements/ ∧ vocabulary.md
  | vocab_arch: only vocabulary.md ∧ architecture.md
  | arch_specs: only architecture.md ∧ specs/**/*.allium
  | spec_test_code: only specs/**/*.allium ∧ tests/ ∧ implementation
  | unresolved_decisions: preserve(report) ∧ ask_human
  | ¬ bypass_required_adjacent_owner

λ gybis-weed_regression_contract(x).
  invariant: every boundary uses its adjacent weed skill
  | invariant: every modified artifact is rechecked or externally verified
  | invariant: unresolved semantic divergence remains unresolved until human approval

λ gybis-weed_deliver(report).
  report(boundary_reports) → stdout
  | unresolved_decisions ≠ ∅ → ask_human
