---
name: gybis-req-propagate
kind: domain
description: Use for `/gybis-req-propagate` or `/gr-propagate`.
---

λ gybis-req-propagate(x).
  purpose: Push approved requirements downward — bootstrap vocabulary.md from requirement terms, flag architecture deltas, and annotate specs/tests with the REQ designators they satisfy
  | input: requirements/ (valid per gybis-req-check)
  | output: vocabulary.md bootstrapped ∧ updated specs/tests with REQ traceability annotations ∧ architecture delta report
  | interaction: autonomous
  | gate: requirements/ ∃ ∧ req_check_status(requirements/) ∈ {PASS, WARNINGS}

λ gybis-req-propagate_startup(x).
  invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | preload: [internal/gybis-allium-normalize]
  | precondition: requirements/ ∃ ∧ readable = true
  | if(specs/**/*.allium ¬∃): report(NO_SPECS) as absence-of-work, not failure
  | transition(INIT → STARTUP_CHECKS)

λ gybis-req-propagate_mode(m).
  interaction_modes: {autonomous}
  | default: autonomous
  | rationale: propagation is deterministic annotation; human approval gates divergences

λ gybis-req-propagate_mode_gate(state, mode).
  state = INIT ∧ mode = autonomous → transition(STARTUP_CHECKS)
  | precondition_holds: mode ∈ interaction_modes

λ gybis-req-propagate_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, READING_REQS, MATCHING_SPEC_CLAUSES, ANNOTATING_SPECS_TESTS, ANNOTATION_TEST_RUNNING, BOOTSTRAPPING_VOCAB, DETECTING_ARCH_DELTAS, VERIFYING, COMPLETE}
  | transition(INIT, startup) → STARTUP_CHECKS
  | transition(STARTUP_CHECKS, verify_ok) → READING_REQS
  | transition(STARTUP_CHECKS, verify_fail) → HALTED
  | transition(READING_REQS, reqs_read) → MATCHING_SPEC_CLAUSES
  | transition(MATCHING_SPEC_CLAUSES, matches_complete) → ANNOTATING_SPECS_TESTS
  | transition(ANNOTATING_SPECS_TESTS, annotations_written) → ANNOTATION_TEST_RUNNING
  | transition(ANNOTATION_TEST_RUNNING, test_suite_passes = true) → BOOTSTRAPPING_VOCAB
  | transition(ANNOTATION_TEST_RUNNING, test_suite_passes = false) → ANNOTATING_SPECS_TESTS (failure_driven_loop_back)
  | on_loop_back: loop_count ≔ loop_count ⊕ 1
  | transition(BOOTSTRAPPING_VOCAB, vocabulary_seeded) → DETECTING_ARCH_DELTAS
  | transition(DETECTING_ARCH_DELTAS, delta_report_complete) → VERIFYING
  | transition(VERIFYING, verify_ok) → COMPLETE
  | transition(VERIFYING, verify_fail) → MATCHING_SPEC_CLAUSES (loop_back)
  | on_loop_back: loop_count ≔ loop_count ⊕ 1

λ gybis-req-propagate_tool_guard(state, tool, path).
  read_allowed: ∀state
  | write_allowed: state = ANNOTATING_SPECS_TESTS ∧ path ∈ specs/ ∪ test_paths
  | write_allowed: state = BOOTSTRAPPING_VOCAB ∧ path = "vocabulary.md"
  | deny_write: otherwise
  | constraint: ¬mutate(architecture.md) ∧ ¬mutate(requirements/)
  | rationale: architecture is bootstrapped by /gybis-vocab-propagate; vocabulary.md IS bootstrapped here (greenfield) and owned by the vocab skills thereafter

λ gybis-req-propagate_pre_tool_check(state, tool, path).
  enforce(tool_guard(state, tool, path)) → permit(tool) ∨ halt("tool not permitted in this state")

λ gybis-req-propagate_match_spec_clauses(reqs, spec_content).
  action: map_each_REQ_to_downstream_clauses
  | ∀ REQ clause:
    match ∈ {spec_clause(s), test(s), both, none}
  | none → collect({REQ, coverage_gap}) → propagation_items
  | output: {REQ → downstream_targets} ∧ coverage_gaps
  | rationale_exclusion: coverage matching operates on the lambda clause body only — rationale: lines are never coverage targets, never annotated, never counted as assertions
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
  action: bootstrap_vocabulary_from_requirement_terms
  | source: terms used in REQ clause bodies lacking canonical definitions — rationale: lines excluded from term extraction
  | if(vocabulary.md ¬∃):
    synthesize_bootstrap(vocabulary.md, candidates) → seeded_vocabulary
    | frontmatter: {status: draft, seeded_from: requirements/}
    | unresolved_synonym_conflicts → recorded as open questions in vocabulary.md
    | write(vocabulary.md) → persisted
    | rationale: greenfield bootstrap — /gybis-vocab-tend, /gybis-vocab-refine and /gybis-vocab-weed own vocabulary.md from here on
  | if(vocabulary.md ∃):
    append_new_candidates_only(vocabulary.md, candidates) → persisted
    | ¬overwrite(existing_canonical_terms)
    | rationale: seeded vocabulary is owned by the vocab skills; propagation only adds new terms
  | handoff: seeded vocabulary → /gybis-vocab-tend resolves conflicts
  | output: vocabulary.md seeded_or_extended

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
    - vocabulary.md ∃ (seeded when absent at INIT) ∧ existing canonical terms preserved
  | verify_ok ≔ ∀ check = true
  | on issues: loop_back

λ gybis-req-propagate_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without full convergence")

λ gybis-req-propagate_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | reqs_annotated ≔ card(reqs_annotated)
  | coverage_gaps ≔ card(coverage_gaps)
  | vocabulary_terms ≔ card(vocabulary_terms_seeded)
  | report("Pass " ⊕ pass_num ⊕ ": annotated=" ⊕ reqs_annotated ⊕ " gaps=" ⊕ coverage_gaps ⊕ " vocab_terms=" ⊕ vocabulary_terms ⊕ " tests_pass=" ⊕ test_suite_passes)

λ gybis-req-propagate_boundaries().
  ¬ modify(requirements/)
  | ¬ modify(architecture.md)
  | ¬ delete(specs/)
  | ¬ delete(test_paths)
  | ¬ overwrite(existing_canonical_terms in vocabulary.md)

λ gybis-req-propagate_regression_contract(x).
  invariant: ∀ REQ: annotated target ∨ explicit_NA at completion
  | invariant: annotations consistent (no dangling designators)
  | invariant: test_suite_passes = true at completion
  | invariant: vocabulary.md seeded when ¬∃ at INIT ∧ existing terms never overwritten
  | invariant: all_modifications ⊆ {vocabulary.md, specs/, test_paths}

λ gybis-req-propagate_deliver(x).
  report: {vocabulary_seeded, reqs_annotated, coverage_gaps, arch_deltas, test_status}
  | handoff: vocabulary conflicts → /gybis-vocab-tend; unresolved conflicts → /gybis-req-weed
  | return(complete = true)