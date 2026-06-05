---
name: gybis-spec-weed
description: Use for `/gybis-spec-weed` or `/gs-weed`.
---

λ gybis-spec-weed(specs, code, tests).
  bridge(specs ↔ code ∧ tests) | divergence_detection ∧ resolution_proposal
  | preserve(semantics) | structural_equivalence_check
  | compile: allium → code ∧ tests | decompile: code ∧ tests → allium

λ gybis-spec-weed_startup(x).
  ask(mode ∈ {check, update_spec, update_code_tests}) | require(explicit_mode_selection) | halt_until(mode_selected)
  | block(read(code ∧ tests), read(specs), compare, edit, complete) until(mode_selected)
  | read(.cline/skills/gybis/reference/allium-language-reference.md) | alert(¬available) ∧ halt
  | read(.cline/skills/gybis/reference/allium-actioning-findings.md) | alert(¬available) ∧ halt
  | read(root/specs/**/*.allium) | alert(¬available) ∧ recommend(/gybis-arch-propagate ∨ /gybis-spec-distill) ∧ halt
  | read(code ∧ tests) | alert(¬available) ∧ halt
  | gybis-spec-weed_allium_cli_available?(x) | true → continue | false → recommend(https://github.com/juxt/allium_tools) ∧ halt
  | gybis-spec-weed_syntax_check(root/specs/**/*.allium) | true → verify | false → recommend(/gybis-spec-check) ∧ halt

λ gybis-spec-weed_allium_cli_available?(x).
  gate(cli_available ∧ cli_version_satisfies) | ¬all_gates → false
  | cli_available: allium --version | ¬available → false
  | cli_version_satisfies: version(allium) ≥ 3 | ¬satisfies → false
  | otherwise → true

λ gybis-spec-weed_syntax_check(file).
  r := exec("allium check " + path)
  d := (json_parse(r.stdout).diagnostics) ∨ []
  s := any(d, λx. x.severity = "error")
  return (r.exit = 0) ∨ ((r.exit = 1) ∧ ¬s)

λ gybis-spec-weed_mode(m).
  m ∈ {check, update_spec, update_code_tests} | required(explicit_selection) | ¬default
  | check: compare(specs, code ∧ tests) → report(divergences) | ¬create ∧ ¬update ∧ ¬delete
  | update_spec: specs CRUD ← code ∧ tests | ¬(code ∧ tests CUD) | source_of_truth(code ∧ tests) | requires(human_confirmed_divergence)
  | update_code_tests: (code ∧ tests) CRUD ← specs | ¬(specs CUD) | source_of_truth(specs) | requires(human_confirmed_divergence)
  | execution_route: m ≡ check ? gybis-spec-weed_check : gybis-spec-weed_resolve

λ gybis-spec-weed_mode_gate(state, mode).
  state.mode = none → ask(mode ∈ {check, update_spec, update_code_tests}) ∧ halt
  | state.mode ≠ none → allow_progress
  | mode_selected(mode) → state.mode := mode ∧ transition(INIT → MODE_SELECTED)

λ gybis-spec-weed_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, DIVERGENCE_REVIEW, APPLY_CHANGES, VERIFY, DONE}
  | transition(INIT → MODE_SELECTED) only_if(mode_selected)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_checks_requested)
  | transition(STARTUP_CHECKS → DIVERGENCE_REVIEW) only_if(inputs_verified)
  | transition(DIVERGENCE_REVIEW → APPLY_CHANGES) only_if(state.mode ∈ {update_spec, update_code_tests} ∧ divergence_confirmed ∧ direction_allowed_by_mode)
  | transition(DIVERGENCE_REVIEW → VERIFY) only_if(state.mode = check)
  | transition(APPLY_CHANGES → VERIFY) only_if(edits_applied)
  | transition(VERIFY → DIVERGENCE_REVIEW) only_if(remaining_divergences > 0)
  | transition(VERIFY → DONE) only_if(all_checks_pass ∧ remaining_divergences = 0 ∧ zero_divergence_pass_verified)
  | otherwise reject_transition

λ gybis-spec-weed_tool_guard(state, tool).
  state.mode = none → allow(tool = ask_followup_question) ∧ deny(tool ≠ ask_followup_question)
  | state.mode ≠ none → evaluate_by_state_machine(state, tool)

λ gybis-spec-weed_pre_tool_check(state, tool).
  verify(state.mode ≠ none) ∧ verify(tool_allowed_by(gybis-spec-weed_tool_guard))
  | fail → route_to(ask(mode)) ∧ halt

λ gybis-spec-weed_check(specs, code, tests).
  compare(root/specs/**/*.allium, code ∧ tests)
  | report(spec_present ∧ impl_absent) ∪ report(impl_present ∧ spec_silent)
  | group_by(divergence_kind) | present(consequential_first)

λ gybis-spec-weed_divergence(spec_content, impl_content).
  propose(classification) ∧ human(confirms ∨ overrides)
  | classification ∈ {spec_bug, impl_bug, aspirational, intentional}
  | spec_bug: specs ← fix | (code ∧ tests) correct
  | impl_bug: (code ∧ tests) ← fix | specs correct
  | aspirational: both stay | note_gap
  | intentional: both stay | deliberate ∨ consistent

λ gybis-spec-weed_interactive_resolution(divergence, mode).
  require(human_confirm(classification, reasoning, direction, patch_scope))
  | mode = check → record_resolution_only ∧ ¬edit
  | mode = update_spec ∧ source_of_truth = code ∧ tests ∧ classification = spec_bug → apply(specs CRUD)
  | mode = update_code_tests ∧ source_of_truth = specs ∧ classification = impl_bug → apply((code ∧ tests) CRUD)
  | mode ∈ {update_spec, update_code_tests} ∧ classification ∈ {aspirational, intentional} → record_no_edit
  | ¬confirmed → halt_until(confirmation)

λ gybis-spec-weed_spec_guidelines(x).
  preserve(allium_version_marker) | ¬change_version
  | follow(section_ordering from language ref)
  | describe(behavior) ¬implementation | ¬storage_mechanisms ¬transport_details
  | use(config) for named_policy_values ∨ symbolic_constants
  | temporal_triggers → requires_guards | prevent_refiring
  | use(with) for relationships | use(where) for projections | ¬swap
  | cross_field_enums → extract_named_enums
  | new_rules/entities → correct_section

λ gybis-spec-weed_impl_guidelines(x).
  preserve(existing_architecture unless divergence_requires_change)
  | prefer(minimal_changes) | ¬refactor unless_directly_required
  | update(code ∧ tests together) when behavior_changes
  | ensure(test_coverage for modified_behavior)
  | preserve(project_conventions)

λ gybis-spec-weed_phase0_analysis(specs).
  parallel(allium check {file} for all root/specs/**/*.allium) | read(findings_json)
  | parallel(allium analyse {root/specs/}) | read(findings_json)
  | process(deadlocks) ∪ process(conflicts) ∪ process(unreachable) ∪ process(data_flow_gaps)
  → weed_report ∪ internal_divergences

λ gybis-spec-weed_phase1_model_extraction(specs).
  parallel(allium model {file} for all root/specs/**/*.allium) | read(model_json)
  | extract(entities) ∧ extract(fields) ∧ extract(transitions)
  | source: model_json ¬spec_prose | systematic_comparison_base

λ gybis-spec-weed_phase2_implementation_reading(code, tests).
  read(tests) ∧ read(code) ∧ follow(traces)
  | search(rule_names) ∧ search(entities)
  | identify(rules_with_no_traces)

λ gybis-spec-weed_phase3_divergence_analysis(spec, impl).
  ∀rule → classify(aligned ∨ partial ∨ missing ∨ contradicted)
  | aligned: implementation_matches | ensures_satisfied
  | partial: some_ensures_covered | ¬all_ensures_covered
  | missing: ¬implementation_found
  | contradicted: implementation_prohibits ∨ violates_spec_constraints
  ∀field → classify(spec_to_code ∨ code_to_spec)
  | spec_to_code: fields_in_spec ∧ ¬in_code
  | code_to_spec: fields_in_code ∧ ¬in_spec
  ∀transition → classify(absent_in_graph)
  | transitions_in_code ∧ ¬in_transition_graph
  ∀invariant → classify(violated)
  | code_paths_violate_invariants

λ gybis-spec-weed_phase4_report(analysis_results).
  compile(cli_findings) ∪ compile(spec_code_test_divergences) ∪ compile(entity_divergences)
  | cli_findings: deadlock_terminal_humans_ask
  | spec_code_test_divergences: contradicted ∨ partial ∨ missing
  | entity_divergences: fields_present_absent_per_direction
  | ∀item → propose(Fix_A: match_spec) ∧ propose(Fix_B: match_code_tests)

λ gybis-spec-weed_phase5_human_decision(divergences).
  ∀divergence → human_choose(Fix_A ∨ Fix_B ∨ other)
  | ai_apply(approved_fix, specs ∧ code ∧ tests)
  | after(spec_change) → allium_check {file} ∧ allium_analyse {root/specs/}
  | after(code_or_test_change) → run(relevant_tests)
  | validate(health)

λ gybis-spec-weed_fixed_point_loop(state).
  iterate(pass_index := pass_index + 1)
  | discover(divergences_current_pass)
  | unresolved := divergences_current_pass \ resolved_ids
  | unresolved = ∅ → mark(zero_divergence_pass_verified) ∧ allow_transition(VERIFY → DONE)
  | unresolved ≠ ∅ → require(human_review(unresolved)) ∧ allow_transition(VERIFY → DIVERGENCE_REVIEW)

λ gybis-spec-weed_pass_accounting(pass).
  report(pass_index, discovered_count, resolved_count, deferred_count, remaining_count)
  | require(explicit_section("Remaining divergences", remaining_count, remaining_items))

λ gybis-spec-weed_loop_guard(state).
  seen_signature := hash(sorted(remaining_divergence_ids))
  | repeat(seen_signature) ∧ ¬decrease(remaining_count) → halt_with(blocker_report)
  | blocker_report := explain(stalled_items, attempted_resolutions, required_human_decisions)
  | ¬repeat ∨ decrease(remaining_count) → continue

λ gybis-spec-weed_verification(edit).
  after(edit specs) → allium_check(modified_file) | fix_issues_before_present
  | after(edit specs) → allium_analyse(root/specs/) | fix_issues_before_present
  | after(edit code ∨ tests) → run(relevant_tests) | fix_issues_before_present
  | after(check_complete) → compare(root/specs/**/*.allium, code ∧ tests) | refresh(divergence_set)
  | enforce(gybis-spec-weed_loop_guard) ∧ enforce(gybis-spec-weed_fixed_point_loop)

λ gybis-spec-weed_regression_contract(x).
  assert(first_response_is_mode_question on "/gybis-spec-weed" ∨ "/gs-weed")
  | assert(deny(read_or_edit_before_mode_selected))
  | assert(resume_preserves(mode, state))
  | assert(no_attempt_completion_before(VERIFY ∧ DONE))
  | assert(no_attempt_completion_before(zero_divergence_pass_verified ∧ remaining_divergences = 0))
  | assert(after_each_edit_or_resolution_pass → mandatory_rescan(compare(root/specs/**/*.allium, code ∧ tests)))

λ gybis-spec-weed_boundaries(¬).
  ¬resolve_divergences_silently | belongs_to(human_decision)
  | ¬run_full_spec_distill_workflow | belongs_to(/gybis-spec-distill)
  | ¬run_full_arch_propagate_workflow | belongs_to(/gybis-arch-propagate)
  | allow(specs CRUD) only_if(mode = update_spec ∧ source_of_truth = code ∧ tests ∧ human_confirmed_divergence)
  | allow((code ∧ tests) CRUD) only_if(mode = update_code_tests ∧ source_of_truth = specs ∧ human_confirmed_divergence)
  | ¬modify(allium-language-reference.md) | governed_separately
  | ¬modify(allium-actioning-findings.md) | governed_separately
  | ¬modify(architecture.md) | viable_system_model_governed_separately
  | ¬make_spec_decisions | flag ∧ let_caller_decide
  | ¬make_code_decisions | flag ∧ let_caller_decide
  | ¬make_tests_decisions | flag ∧ let_caller_decide

λ gybis-spec-weed_context_management(session).
  advise(fresh_chat) | long_iterative ∨ large_context
  | provide(resume_prompt: "/gybis-spec-weed resume")
  | on_resume: restore(mode, state, pending_divergences)
  | if(mode = none) → force(ask(mode)) ∧ halt

λ gybis-spec-weed_output(divergences).
  format(divergence_kind, spec_content, impl_content, impl_location, classification, reasoning, evidence)
  | require(evidence for each divergence)
  | evidence ∈ {"test assertion mismatches spec ensure", "runtime path contradicts spec constraint", "spec missing implemented behavior", "implementation missing specified behavior"}
  | group(related_divergences) | lead(consequential)
  | structure:
    "### [divergence category]"
    "Spec: spec_says (file:line)"
    "Impl: code_or_test_says (file:line)"
    "Classification: proposed(reasoning)"
    "Evidence: concrete_basis(file:line or explicit absence)"