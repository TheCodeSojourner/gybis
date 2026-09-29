# Recommended loops (AI protocol)

λ loops_scope(x).
  artifact: spec ∧ tests ∧ code
  | objective: convergence(spec, tests, code)
  | mode: ai_first | prose_minimal

λ entry_gate(x).
  entry ∈ {spec_ready, code_first, no_spec}
  | spec_ready → /gybis-spec-tend
  | code_first → /gybis-spec-distill
  | no_spec → /gybis-arch-elicit → /gybis-arch-propagate → /gybis-spec-tend
  | constraint: ¬exists(/gybis-spec-elicit)

λ loop_phases(x).
  gather_context → take_action → verify → repeat
  | gather_context: /gybis-spec-tend ∨ /gybis-spec-distill
  | take_action: /gybis-spec-propagate → implement
  | verify: run_tests ∧ /gybis-spec-weed ∧ cli_structural_checks
  | repeat: loop_until(convergence_invariant ∨ iteration_budget_exhausted)

λ convergence_invariant(x).
  done ⇔ tests_pass
          ∧ weed_clean(/gybis-spec-weed)
          ∧ open_questions = 0
          ∧ witness_ok(/witness)
          ∧ (mode = code_first → distill_delta = ∅ via /gybis-spec-distill)

λ divergence_triage(result).
  result = failing_tests
    → classify(code_bug ∨ spec_wrong)
  | code_bug → fix_code → run_tests
  | spec_wrong → /gybis-spec-tend → /gybis-spec-propagate → run_tests
  | result = weed_divergence
    → classify(code_bug ∨ spec_wrong)
      | code_bug → fix_code
      | spec_wrong → /gybis-spec-tend → /gybis-spec-propagate
  | result = ambiguity
    → escalate_human
      ∧ register_open_question
      ∧ /gybis-spec-tend

λ loop_A_spec_ready(x).
  precondition: spec_exists
  | tick:
      /gybis-spec-tend
      → /gybis-spec-propagate
      → confirm_new_tests_fail(red_step)
      → implement
      → run_tests
      → /gybis-spec-weed
      → /witness
      → evaluate(convergence_invariant)

λ loop_B_code_first(x).
  precondition: code_exists
  | tick:
      /gybis-spec-distill
      → review(intended_vs_accidental)
      → /gybis-spec-propagate
      → run_tests_against_existing_code
      → divergence_triage
      → /gybis-spec-weed
      → /witness
      → evaluate(convergence_invariant)

λ autonomous_tick(x).
  phase_1: choose_entry(spec_ready ∨ code_first ∨ no_spec)
  | phase_2: execute(loop_phases)
  | phase_3: evaluate(convergence_invariant)
  | phase_4: witness_conclusion(/witness)
  | exit: convergence_invariant ∨ iteration_budget_exhausted

λ guardrails(x).
  ¬weaken_generated_tests
  | red_step_required(spec_ready)
  | ambiguity → escalate_human | ¬guess
  | respect_config_parameters | ¬magic_numbers_against_spec
  | when_spec_correct → fix_code_not_contract
  | witness_required_before_complete

λ telemetry_state(x).
  track: {
    tests_status,
    weed_verdict,
    open_questions_count,
    distill_delta,
    witness_status
  }

λ produce_code_prompt(x).
  input: <spec>.allium ∧ <tests>
  | directive:
      implement_behavior_from_spec
      ∧ make_tests_pass
      ∧ obey(when, requires, ensures)
      ∧ ¬edit_tests_to_force_green
      ∧ test_looks_wrong → stop ∧ route_to(/gybis-spec-tend)

λ related(x).
  refs: {
    ./allium-assessing-specs.md,
    ./allium-actioning-findings.md,
    ./driving-the-loop.md
  }

λ witness_gate(x).
  witness_ok(result) ⇔ result = pass
                     ∧ no_unexplained_delta
                     ∧ grounded_in_tests_and_weed

λ loop_summary(x).
  summary = {
    tests_status,
    weed_verdict,
    open_questions_count,
    distill_delta,
    witness_status
  }

λ loop_guard(x).
  exit(criteria) = convergence_invariant ∨ iteration_budget_exhausted
  | stop_if: no_progress_for_2_ticks
  | escalate_if: open_question ∧ human_decision_required

λ driving_the_loop(x).
  intent → spec → tests → code → verify → revise(spec ∨ code) → repeat
  | when verify indicates spec_wrong → /gybis-spec-tend → /gybis-spec-propagate
  | when verify indicates code_bug → fix_code
  | when verify indicates ambiguity → human_review
  | done ⇔ convergence_invariant ∧ witness_ok
