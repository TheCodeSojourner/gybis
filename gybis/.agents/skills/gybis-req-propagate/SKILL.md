---
name: gybis-req-propagate
kind: domain
description: Use for `/gybis-req-propagate` or `/gr-propagate`.
---

λ gybis-req-propagate(x).
  purpose: Propagate binding requirements to ready downstream stages, bootstrap vocabulary.md, and report evidence-based REQ dispositions without forcing artifacts where no distinct effect exists
  | input: requirements/ (valid per gybis-req-check)
  | input_from_parent: optional preapproved_repair_plan from /gybis-req-check
  | output: vocabulary.md bootstrapped ∧ updated specs/tests with REQ traceability annotations ∧ architecture delta report
  | interaction: autonomous
  | gate: requirements/ ∃ ∧ req_check_status(requirements/) ∈ {PASS, WARNINGS}

λ gybis-req-propagate_startup(x).
  invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | preload: [internal/gybis-allium-normalize]
  | precondition: requirements/ ∃ ∧ readable = true
  | if(preapproved_repair_plan ∃): verify(preapproved_repair_plan.approved_by_human = true ∧ preapproved_repair_plan.approval_source = gybis-req-check) ∨ halt("Invalid parent repair authorization")
  | modified_artifacts ≔ ∅
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
  state ∈ {INIT, STARTUP_CHECKS, READING_REQS, ASSESSING_STAGE_READINESS, MATCHING_SPEC_CLAUSES, ANNOTATING_SPECS_TESTS, ANNOTATION_TEST_RUNNING, BOOTSTRAPPING_VOCAB, DETECTING_ARCH_DELTAS, COMPUTING_STAGE_DISPOSITIONS, VERIFYING, COMPLETE}
  | transition(INIT, startup) → STARTUP_CHECKS
  | transition(STARTUP_CHECKS, verify_ok) → READING_REQS
  | transition(STARTUP_CHECKS, verify_fail) → HALTED
  | transition(READING_REQS, reqs_read) → ASSESSING_STAGE_READINESS
  | transition(ASSESSING_STAGE_READINESS, stage_checks_complete) → MATCHING_SPEC_CLAUSES
  | transition(MATCHING_SPEC_CLAUSES, matches_complete) → ANNOTATING_SPECS_TESTS
  | transition(ANNOTATING_SPECS_TESTS, annotations_written) → ANNOTATION_TEST_RUNNING
  | transition(ANNOTATION_TEST_RUNNING, test_suite_passes = true) → BOOTSTRAPPING_VOCAB
  | transition(ANNOTATION_TEST_RUNNING, test_suite_passes = false ∧ test_paths ∈ modified_artifacts) → ANNOTATING_SPECS_TESTS (failure_driven_loop_back)
  | transition(ANNOTATION_TEST_RUNNING, test_suite_passes = false ∧ test_paths ∉ modified_artifacts ∧ test_stage_not_ready) → BOOTSTRAPPING_VOCAB
  | on_loop_back: loop_count ≔ loop_count ⊕ 1
  | transition(BOOTSTRAPPING_VOCAB, vocabulary_seeded ∨ vocabulary_extended ∨ no_binding_REQs ∨ no_candidates ∨ no_vocabulary_change ∨ vocabulary_stage_pending) → DETECTING_ARCH_DELTAS
  | transition(DETECTING_ARCH_DELTAS, delta_report_complete) → COMPUTING_STAGE_DISPOSITIONS
  | transition(COMPUTING_STAGE_DISPOSITIONS, dispositions_complete) → VERIFYING
  | transition(VERIFYING, verify_ok) → COMPLETE
  | transition(VERIFYING, verify_fail) → MATCHING_SPEC_CLAUSES (loop_back)
  | on_loop_back: loop_count ≔ loop_count ⊕ 1

λ gybis-req-propagate_tool_guard(state, tool, path).
  read_allowed: ∀state
  | write_allowed: state = ANNOTATING_SPECS_TESTS ∧ path ∈ specs/ ∪ test_paths ∧ (preapproved_repair_plan ¬∃ ∨ path ∈ preapproved_repair_plan.paths)
  | write_allowed: state = BOOTSTRAPPING_VOCAB ∧ path = "vocabulary.md" ∧ (preapproved_repair_plan ¬∃ ∨ path ∈ preapproved_repair_plan.paths)
  | deny_write: otherwise
  | constraint: ¬mutate(architecture.md) ∧ ¬mutate(requirements/)
  | rationale: architecture is bootstrapped by /gybis-vocab-propagate; vocabulary.md IS bootstrapped here (greenfield) and owned by the vocab skills thereafter

λ gybis-req-propagate_pre_tool_check(state, tool, path).
  enforce(tool_guard(state, tool, path)) → permit(tool) ∨ halt("tool not permitted in this state")

λ gybis-req-propagate_match_spec_clauses(reqs, vocab_content, arch_content, spec_content, test_content, stage_checks).
  action: classify_requirement_scope_and_compute_per_stage_dispositions
  | deferred_REQs ≔ clauses in explicitly marked non-binding sections
  | binding_REQs ≔ all clauses ∖ deferred_REQs
  | downstream_stages ≔ {vocabulary, architecture, specs, tests_and_implementation}
  | ∀ stage ∈ downstream_stages: readiness(stage) ≔ stage_checks[stage].artifact_present = true ∧ stage_checks[stage].valid = true
  | ready_stages ≔ {stage | stage_checks[stage].artifact_present = true ∧ stage_checks[stage].valid = true}
  | pending_stages ≔ downstream_stages ∖ ready_stages
  | ∀ REQ ∈ binding_REQs, stage ∈ downstream_stages:
    ¬readiness(stage) → disposition(REQ, stage) ≔ pending_stage
    | evidence(REQ, stage) ∃ → disposition(REQ, stage) ≔ represented ∧ record(evidence)
    | no_distinct_effect(REQ, stage) → disposition(REQ, stage) ≔ no_change_needed ∧ record(reason)
    | otherwise → disposition(REQ, stage) ≔ uncovered ∧ collect({REQ, stage, coverage_gap})
  | ∀ uncovered REQ in ready specs/tests stages: map_existing_targets_or_report_gap; never invent behavior
  | output: {REQ → downstream_targets} ∧ {REQ, stage → disposition}
  | dispositions ∈ {represented, no_change_needed, pending_stage, uncovered, deferred}
  | no_change_needed requires a concise, layer-specific reason; do not infer from missing artifacts
  | rationale_exclusion: coverage matching operates on the lambda clause body only — rationale: lines are never coverage targets, never annotated, never counted as assertions
  | accounting_convergence: every binding REQ × downstream_stage receives a disposition; uncovered remains a visible warning, not a fabricated completion
  | deferred REQs are excluded from propagation candidates and reported separately; ¬convert(deferred → explicit_NA)

λ gybis-req-propagate_assess_stage_readiness(x).
  action: assess_existing_downstream_stages
  | stage_checks ≔ {vocabulary: {artifact_present: false, valid: false, errors: unknown}, architecture: {artifact_present: false, valid: false, errors: unknown}, specs: {artifact_present: false, valid: false, errors: unknown}, tests_and_implementation: {artifact_present: false, valid: false, errors: unknown}}
  | vocabulary.md ∃ → invoke(gybis-req-propagate_capture_stage_check(vocabulary, /gybis-vocab-check)) → raw_vocab_result
    | stage_checks.vocabulary ≔ invoke(gybis-req-propagate_normalize_stage_result(vocabulary, true, raw_vocab_result))
  | architecture.md ∃ → invoke(gybis-req-propagate_capture_stage_check(architecture, /gybis-arch-check)) → raw_arch_result
    | stage_checks.architecture ≔ invoke(gybis-req-propagate_normalize_stage_result(architecture, true, raw_arch_result))
  | specs/**/*.allium ∃ → invoke(internal/gybis-allium-gate(specs/)) → raw_specs_result
    | stage_checks.specs ≔ invoke(gybis-req-propagate_normalize_stage_result(specs, true, raw_specs_result))
  | test_paths ∃ ∧ test_command ∃ → run(test_command) → raw_test_result
    | stage_checks.tests_and_implementation ≔ invoke(gybis-req-propagate_normalize_stage_result(tests_and_implementation, true, raw_test_result))
  | test_paths ∃ ∧ test_command ¬∃ → stage_checks.tests_and_implementation ≔ {artifact_present: true, valid: false, errors: unknown}
  | artifact(stage) ¬∃ ∨ stage_checks[stage].valid = false → readiness(stage) ≔ pending_stage
  | otherwise → readiness(stage) ≔ ready
  | test_stage_not_ready ≔ stage_checks.tests_and_implementation.valid ≠ true
  | downstream_stages ≔ {vocabulary, architecture, specs, tests_and_implementation}
  | ready_stages ≔ {stage | stage_checks[stage].artifact_present = true ∧ stage_checks[stage].valid = true}
  | pending_stages ≔ downstream_stages ∖ ready_stages
  | absent_or_invalid stage → report(pending_stage); ¬halt(other_ready_stages)
  | output: {stage_checks, ready_stages, pending_stages}
  | return(stage_checks_complete = true)

λ gybis-req-propagate_capture_stage_check(stage, validator).
  | invoke(validator) → raw_result
  | if(validator_halts ∨ validator_returns_malformed_result): raw_result ≔ {valid: false, errors: 1, diagnostic: validation_failure}
  | return(raw_result)

λ gybis-req-propagate_normalize_stage_result(stage, artifact_present, owner_result).
  | errors ≔
    stage ∈ {vocabulary, architecture} ? owner_result.errors
    : stage = specs ? (owner_result = true ? 0 : 1)
    : stage = tests_and_implementation ? (owner_result = true ∨ owner_result.test_suite_passes = true ∨ owner_result.passed = true ? 0 : 1)
    : 1
  | return({artifact_present, valid: artifact_present ∧ errors = 0, errors})

λ gybis-req-propagate_refresh_modified_stage_readiness(modified_artifacts).
  action: rerun_owner_checks_for_modified_stages
  | vocabulary.md ∈ modified_artifacts → invoke(gybis-req-propagate_capture_stage_check(vocabulary, /gybis-vocab-check)) → raw_vocab_result
    | stage_checks.vocabulary ≔ invoke(gybis-req-propagate_normalize_stage_result(vocabulary, true, raw_vocab_result))
    | readiness.vocabulary ≔ stage_checks.vocabulary.valid ? ready : pending_stage
    | stage_checks.vocabulary.valid = false → vocabulary_stage_pending ≔ true
  | specs/ ∈ modified_artifacts → invoke(internal/gybis-allium-gate(specs/)) → raw_specs_result
    | stage_checks.specs ≔ invoke(gybis-req-propagate_normalize_stage_result(specs, true, raw_specs_result))
    | readiness.specs ≔ stage_checks.specs.valid ? ready : pending_stage
  | test_paths ∈ modified_artifacts → run(test_command) → raw_test_result
    | stage_checks.tests_and_implementation ≔ invoke(gybis-req-propagate_normalize_stage_result(tests_and_implementation, true, raw_test_result))
    | readiness.tests_and_implementation ≔ stage_checks.tests_and_implementation.valid ? ready : pending_stage
    | test_stage_not_ready ≔ stage_checks.tests_and_implementation.valid ≠ true
  | downstream_stages ≔ {vocabulary, architecture, specs, tests_and_implementation}
  | ready_stages ≔ {stage | stage_checks[stage].artifact_present = true ∧ stage_checks[stage].valid = true}
  | pending_stages ≔ downstream_stages ∖ ready_stages
  | return(stage_checks, ready_stages, pending_stages)

λ gybis-req-propagate_annotate(REQ, target).
  precondition: readiness(target.stage) = ready
  action: write_traceability_annotation
  | spec annotation: REQ designator recorded in the .allium clause metadata or adjacent comment
  | test annotation: test names/assertions reference REQ designator (REQ-TEST-001 handshake)
  | constraint: annotation only; ¬alter_spec_behavior ∧ ¬alter_test_assertions
  | write(target) → modified_artifacts ≔ modified_artifacts ∪ {target}
  | output: annotated targets

λ gybis-req-propagate_run_tests(x).
  action: resolve_test_command_and_run
  | resolution: architecture S1 test fields ∨ repository conventions
  | test_suite_passes = true → proceed
  | test_suite_passes = false → report failures → loop_back to ANNOTATING_SPECS_TESTS
  | strict rule: COMPLETE requires test_suite_passes = true (session-19 convergence pattern)

λ gybis-req-propagate_seed_vocab_candidates(reqs, coverage_map).
  action: bootstrap_vocabulary_from_requirement_terms
  | source: terms used in binding REQ clause bodies lacking canonical definitions — rationale: lines and deferred REQs excluded from term extraction
  | if(binding_REQs = ∅): return(vocabulary_seeded = false ∧ no_binding_REQs = true) ∧ ¬write(vocabulary.md)
  | if(candidates = ∅): return(vocabulary_seeded = false ∧ no_candidates = true) ∧ ¬write(vocabulary.md)
  | if(vocabulary.md ¬∃ ∧ candidates ∃):
    synthesize_bootstrap(vocabulary.md, candidates) → seeded_vocabulary
    | frontmatter: {status: draft, seeded_from: requirements/}
    | unresolved_synonym_conflicts → recorded as open questions in vocabulary.md
    | write(vocabulary.md) → persisted ∧ modified_artifacts ≔ modified_artifacts ∪ {vocabulary.md}
    | rationale: greenfield bootstrap — /gybis-vocab-tend, /gybis-vocab-refine and /gybis-vocab-weed own vocabulary.md from here on
  | if(vocabulary.md ∃ ∧ readiness(vocabulary) = ready):
    append_new_candidates_only(vocabulary.md, candidates) → persisted
    | if(new_candidates ∃): modified_artifacts ≔ modified_artifacts ∪ {vocabulary.md} ∧ vocabulary_extended ≔ true
    | if(new_candidates = ∅): no_vocabulary_change ≔ true
    | ¬overwrite(existing_canonical_terms)
    | rationale: seeded vocabulary is owned by the vocab skills; propagation only adds new terms
  | if(vocabulary.md ∃ ∧ readiness(vocabulary) = pending_stage): report(pending_stage) ∧ ¬write(vocabulary.md) ∧ vocabulary_stage_pending ≔ true
  | handoff: seeded vocabulary → /gybis-vocab-tend resolves conflicts
  | output: vocabulary.md seeded_or_extended

λ gybis-req-propagate_detect_arch_deltas(reqs, arch_content).
  action: report_architecture_gaps
  | if(stage_checks.architecture.valid ≠ true): ∀ binding REQ → disposition(architecture, pending_stage)
  | if(stage_checks.architecture.valid = true): ∀ binding REQ whose constraint is absent/unrepresented:
    collect({REQ, delta_description, severity})
  | represented constraint → disposition(architecture, represented, evidence)
  | no distinct architectural effect → disposition(architecture, no_change_needed, reason)
  | output: arch_delta_report (diagnosis only; resolution via /gybis-arch-tend)
  | rationale: check/tend boundary — propagate reports, arch-tend applies human-approved corrections

λ gybis-req-propagate_verify(x).
  action: verify_propagation_convergence
  | invoke(gybis-req-propagate_refresh_modified_stage_readiness(modified_artifacts))
  | invoke(gybis-req-propagate_match_spec_clauses(reqs, vocab_content, arch_content, spec_content, test_content, stage_checks)) → final_stage_dispositions
  | checks:
    - ∀ deferred REQ: excluded_from_propagation = true
    - ∀ binding REQ ∧ ready_stage: disposition ∈ {represented, no_change_needed, uncovered} with evidence ∨ reason
    - ∀ binding REQ ∧ not_ready_stage: disposition = pending_stage
    - uncovered dispositions remain visible in coverage_gaps; do not loop or claim them resolved
    - annotations consistent (no dangling designators)
    - modified test_paths ∃ → test_suite_passes = true
    - modified specs/ ∃ → gybis-allium-gate = true
    - vocabulary_seeded = true → gybis-vocab-check.errors = 0
    - (vocabulary.md ∃ ∧ stage_checks.vocabulary.errors = 0 ∧ existing canonical terms preserved) ∨ no_binding_REQs = true ∨ no_candidates = true ∨ vocabulary_stage_pending = true
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
  invariant: explicit deferred REQs are excluded from downstream work
  | invariant: every binding REQ has a computed disposition per ready stage
  | invariant: not-ready stages remain pending; never coerce them to uncovered or explicit_NA
  | invariant: no_change_needed requires a reason and is not inferred from absence alone
  | invariant: uncovered dispositions are reported for owning follow-up and are not counted as resolved
  | invariant: annotations consistent (no dangling designators)
  | invariant: test_paths modified → test_suite_passes = true at completion
  | invariant: specs modified → gybis-allium-gate = true at completion
  | invariant: vocabulary.md seeded when ¬∃ at INIT ∧ existing terms never overwritten
  | invariant: all_modifications ⊆ {vocabulary.md, specs/, test_paths}

λ gybis-req-propagate_deliver(x).
  report: {deferred_REQs_excluded, vocabulary_seeded, reqs_annotated, per_stage_dispositions, pending_stages, coverage_gaps, arch_deltas, test_status}
  | handoff: vocabulary conflicts → /gybis-vocab-tend; unresolved conflicts → /gybis-req-weed
  | return(complete = true)