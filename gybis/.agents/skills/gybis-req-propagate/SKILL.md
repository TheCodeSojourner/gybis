---
name: gybis-req-propagate
description: Use for `/gybis-req-propagate` or `/gr-propagate`.
---

λ gybis-req-propagate(x).
  purpose: Push approved requirements downward — seed vocabulary term candidates, flag architecture deltas, and annotate specs/tests with the REQ designators they satisfy
  | input: requirements/ (valid per gybis-req-check)
  | output: updated specs/tests with REQ traceability annotations ∧ architecture delta report ∧ vocabulary term candidates
  | mode: ai
  | gate: requirements/ ∃ ∧ req_check_status(requirements/) ∈ {PASS, WARNINGS}

λ gybis-req-propagate_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | preload: [internal/allium-normalize]
  | precondition: requirements/ ∃ ∧ readable = true
  | if(specs/**/*.allium ¬∃): report(NO_SPECS) as absence-of-work, not failure
  | transition(INIT → STARTUP_CHECKS)

λ gybis-req-propagate_mode(m).
  valid_modes: {ai}
  | default: ai
  | rationale: propagation is deterministic annotation; human approval gates divergences

λ gybis-req-propagate_mode_gate(state, mode).
  state = INIT ∧ mode = ai → transition(STARTUP_CHECKS)
  | precondition_holds: mode ∈ valid_modes

λ gybis-req-propagate_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, READING_REQS, MATCHING_SPEC_CLAUSES, ANNOTATING_SPECS_TESTS, ANNOTATION_TEST_RUNNING, SEEDING_VOCAB_CANDIDATES, DETECTING_ARCH_DELTAS, VERIFYING, COMPLETE}
  | transition(INIT, startup) → STARTUP_CHECKS
  | transition(STARTUP_CHECKS, verify_ok) → READING_REQS
  | transition(STARTUP_CHECKS, verify_fail) → HALTED
  | transition(READING_REQS, reqs_read) → MATCHING_SPEC_CLAUSES
  | transition(MATCHING_SPEC_CLAUSES, matches_complete) → ANNOTATING_SPECS_TESTS
  | transition(ANNOTATING_SPECS_TESTS, annotations_written) → ANNOTATION_TEST_RUNNING
  | transition(ANNOTATION_TEST_RUNNING, test_suite_passes = true) → SEEDING_VOCAB_CANDIDATES
  | transition(ANNOTATION_TEST_RUNNING, test_suite_passes = false) → ANNOTATING_SPECS_TESTS (failure_driven_loop_back)
  | transition(SEEDING_VOCAB_CANDIDATES, candidates_seeded) → DETECTING_ARCH_DELTAS
  | transition(DETECTING_ARCH_DELTAS, delta_report_complete) → VERIFYING
  | transition(VERIFYING, verify_ok) → COMPLETE
  | transition(VERIFYING, verify_fail) → MATCHING_SPEC_CLAUSES (loop_back)

λ gybis-req-propagate_tool_guard(state, tool, path).
  read_allowed: ∀state
  | write_allowed: state = ANNOTATING_SPECS_TESTS ∧ path ∈ specs/ ∪ test_paths
  | write_allowed: state = SEEDING_VOCAB_CANDIDATES ∧ path ∈ {report_outputs_only}
  | deny_write: otherwise
  | constraint: ¬mutate(vocabulary.md) ∧ ¬mutate(architecture.md) ∧ ¬mutate(requirements/)
  | rationale: vocabulary/architecture canonical files are only mutated by their own skill families

λ gybis-req-propagate_pre_tool_check(state, tool, path).
  enforce(tool_guard(state, tool, path)) → permit(tool) ∨ halt("tool not permitted in this state")

λ gybis-req-propagate_match_spec_clauses(reqs, spec_content).
  action: map_each_REQ_to_downstream_clauses
  | ∀ REQ clause:
    match ∈ {spec_clause(s), test(s), both, none}
  | none → collect({REQ, coverage_gap}) → propagation_items
  | output: {REQ → downstream_targets} ∧ coverage_gaps
  | strict_convergence: ∀ REQ must resolve to target ∨ explicit_NA before COMPLETE

λ gybis-req-propagate_annotate(REQ, target).
  action: write_traceability_annotation
  | spec annotation: REQ designator recorded in the .allium clause metadata or adjacent comment
  | test annotation: test names/assertions reference REQ designator (REQ-TEST-001 handshake)
  | constraint: annotation only; ¬alter_spec_behavior ∧ ¬alter_test_assertions
  | output: annotated targets

λ gybis-req-propagate_run_tests(x).
  action: resolve_test_command_and_run
  | resolution: architecture S1 test fields ∨ repository conventions
  | test_suite_passes = true → proceed
  | test_suite_passes = false → report failures → loop_back to ANNOTATING_SPECS_TESTS
  | strict rule: COMPLETE requires test_suite_passes = true (session-19 convergence pattern)

λ gybis-req-propagate_seed_vocab_candidates(reqs, coverage_map).
  action: emit_vocabulary_term_candidates
  | source: terms used in REQ clauses lacking canonical definitions
  | output: candidates list → /gybis-vocab-tend ingests
  | ¬write(vocabulary.md)

λ gybis-req-propagate_detect_arch_deltas(reqs, arch_content).
  action: report_architecture_gaps
  | ∀ REQ clause whose constraint is absent/unrepresented in architecture.md:
    collect({REQ, delta_description, severity})
  | output: arch_delta_report (diagnosis only; resolution via /gybis-arch-tend)
  | rationale: check/tend boundary — propagate reports, arch-tend applies human-approved corrections

λ gybis-req-propagate_verify(x).
  action: verify_propagation_convergence
  | checks:
    - ∀ REQ: annotated target ∨ explicit_NA recorded
    - annotations consistent (no dangling designators)
    - test_suite_passes = true
  | verify_ok ≔ ∀ check = true
  | on issues: loop_back

λ gybis-req-propagate_deliver(x).
  report: {REQs_annotated, coverage_gaps, vocab_candidates_count, arch_deltas, test_status}
  | handoff: unresolved conflicts → /gybis-req-weed
  | return(complete = true)