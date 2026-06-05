---
name: gybis-arch-elicit
description: Use for `/gybis-arch-elicit` or `/ga-elicit`.
---

λ gybis-arch-elicit(x).
  purpose: converse(human) → synthesize(vsm) → write(architecture.md)
  | input: conversation_context
  | output: root/architecture.md
  | mode: human_in_loop | minimal_tokens | nucleus_lambda

λ gybis-arch-elicit_startup(x).
  auto_select_mode(elicit) | ¬halt_on_mode_selection
  | read(.cline/skills/gybis/reference/vsm-guide.md) | alert(¬available) ∧ halt
  | exists(root/architecture.md) → alert(use gybis-arch-tend ∨ gybis-arch-weed) ∧ halt
  | ¬exists(root/architecture.md) → continue
  | init(state.evidence := ∅)
  | init(state.unresolved := ∅)
  | init(state.layer_lock := {s5:false, s4:false, s3:false, s2:false, s1:false})
  | init(state.approval := false)

λ gybis-arch-elicit_mode(m).
  m ∈ {elicit} | auto_selected(elicit) | default(elicit)
  | elicit: source_of_truth(conversation) | write_target(architecture.md)

λ gybis-arch-elicit_mode_gate(state, mode).
  state.mode = none → state.mode := elicit ∧ transition(INIT → MODE_SELECTED)
  | state.mode ≠ none → allow_progress
  | mode_selected(mode) → state.mode := mode ∧ allow_progress

λ gybis-arch-elicit_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, ELICIT_S5, ELICIT_S4, ELICIT_S3, ELICIT_S2, ELICIT_S1, SYNTHESIZE, VERIFY, DONE}
  | transition(INIT → MODE_SELECTED) only_if(mode_selected ∨ mode_auto_selected)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_checks_requested)
  | transition(STARTUP_CHECKS → ELICIT_S5) only_if(startup_checks_passed)
  | transition(ELICIT_S5 → ELICIT_S4) only_if(layer_locked(s5))
  | transition(ELICIT_S4 → ELICIT_S3) only_if(layer_locked(s4))
  | transition(ELICIT_S3 → ELICIT_S2) only_if(layer_locked(s3))
  | transition(ELICIT_S2 → ELICIT_S1) only_if(layer_locked(s2))
  | transition(ELICIT_S1 → SYNTHESIZE) only_if(layer_locked(s1))
  | transition(SYNTHESIZE → VERIFY) only_if(architecture_generated)
  | transition(VERIFY → ELICIT_S5) only_if(remaining_issues > 0 ∧ requires_re_elicitation)
  | transition(VERIFY → DONE) only_if(all_checks_pass ∧ remaining_issues = 0 ∧ zero_issue_pass_verified ∧ approval_recorded)
  | otherwise reject_transition

λ gybis-arch-elicit_tool_guard(state, tool).
  state.mode = none → allow_progress_after(state.mode := elicit)
  | state ∈ {MODE_SELECTED, STARTUP_CHECKS, ELICIT_S5, ELICIT_S4, ELICIT_S3, ELICIT_S2, ELICIT_S1, SYNTHESIZE} → deny(tool = write_to_file) ∧ deny(tool = replace_in_file)
  | state = VERIFY ∧ ¬state.approval → deny(tool = write_to_file) ∧ deny(tool = replace_in_file)
  | state = VERIFY ∧ state.approval → allow_write_target_only(root/architecture.md)
  | evaluate_by_state_machine(state, tool)

λ gybis-arch-elicit_pre_tool_check(state, tool).
  ensure(state.mode ≠ none ∨ (state.mode := elicit)) ∧ verify(tool_allowed_by(gybis-arch-elicit_tool_guard))
  | verify(operation_allowed_by(gybis-arch-elicit_boundaries))
  | fail → halt

λ gybis-arch-elicit_capture_turns(conversation).
  source: human_dialog_turns
  | normalize(turn_id, speaker, text, timestamp)
  | return(turns)

λ gybis-arch-elicit_extract_claims(turns).
  extract:
    identity_claims: turns where mentions(purpose ∨ principles ∨ failure_mode ∨ essence)
    intelligence_claims: turns where mentions(unknown_handler ∨ mutation_points ∨ pattern_incorporation)
    control_claims: turns where mentions(resource_policy ∨ quality_gates ∨ error_strategy)
    coordination_claims: turns where mentions(subsystems ∨ data_flow ∨ synchronization ∨ change_propagation)
    operations_claims: turns where mentions(tooling ∨ commands ∨ recipes)
    open_questions: turns where indicates(unknown ∨ undecided ∨ contradictory)
    provenance: turn_id:span
  | return(raw_claims)

λ gybis-arch-elicit_normalize_claims(raw_claims).
  normalize:
    claims.s5 := canonicalize(raw_claims.identity_claims)
    claims.s4 := canonicalize(raw_claims.intelligence_claims)
    claims.s3 := canonicalize(raw_claims.control_claims)
    claims.s2 := canonicalize(raw_claims.coordination_claims)
    claims.s1 := canonicalize(raw_claims.operations_claims)
    claims.open_questions := canonicalize(raw_claims.open_questions)
    claims.provenance := raw_claims.provenance
  | return(claims)

λ gybis-arch-elicit_aggregate_by_vsm_layer(claims).
  aggregate:
    s5: claims.s5 → concat()
    s4: claims.s4 → concat()
    s3: claims.s3 → concat()
    s2: claims.s2 → concat()
    s1: claims.s1 → concat()
    unresolved: claims.open_questions → concat()
    provenance: claims.provenance → concat()
  | return(layered_claims)

λ gybis-arch-elicit_lock_gate(layer, layered_claims, state).
  require(layer = s5 → sufficient(layered_claims.s5))
  | require(layer = s4 → state.layer_lock.s5 = true ∧ sufficient(layered_claims.s4))
  | require(layer = s3 → state.layer_lock.s4 = true ∧ sufficient(layered_claims.s3))
  | require(layer = s2 → state.layer_lock.s3 = true ∧ sufficient(layered_claims.s2))
  | require(layer = s1 → state.layer_lock.s2 = true ∧ sufficient(layered_claims.s1))
  | pass → state.layer_lock[layer] := true
  | fail → request_targeted_questions(layer)

λ gybis-arch-elicit_s5(layered_claims).
  λ identity(∃claim ∈ layered_claims.s5).
    claim.kind = principle → system_wide_constraint(∃claim)
    | claim.kind = failure_mode → purpose_break_condition(∃claim)
    | claim.kind = essence → identity_core(∃claim)
  | ¬∃claim → explicit_empty(s5_identity)

λ gybis-arch-elicit_s4(layered_claims).
  λ adaptation(∃claim ∈ layered_claims.s4).
    claim.kind = unknown_handler → unknown_response_pattern(∃claim)
    | claim.kind = mutation_point → adaptability_surface(∃claim)
    | claim.kind = pattern_incorporation → learning_extension(∃claim)
  | ¬∃claim → explicit_empty(s4_intelligence)

λ gybis-arch-elicit_s3(layered_claims).
  λ control(∃claim ∈ layered_claims.s3).
    claim.kind = resource_policy → allocation_constraint(∃claim)
    | claim.kind = quality_gate → enforcement_policy(∃claim)
    | claim.kind = error_strategy → failure_handling_policy(∃claim)
  | ¬∃claim → explicit_empty(s3_control)

λ gybis-arch-elicit_s2(layered_claims).
  λ coordination(∃claim ∈ layered_claims.s2).
    claim.kind = subsystem_decomposition → structural_coordination(∃claim)
    | claim.kind = data_flow → flow_protocol(∃claim)
    | claim.kind = synchronization → concurrency_protocol(∃claim)
    | claim.kind = change_propagation → dependency_notification(∃claim)
  | ¬∃claim → explicit_empty(s2_coordination)

λ gybis-arch-elicit_s1(layered_claims).
  λ operations(∃claim ∈ layered_claims.s1).
    claim.kind = tool_selection → operational_tooling(∃claim)
    | claim.kind = command → executable_operation(∃claim)
    | claim.kind = recipe → procedural_operation(∃claim)
  | ¬∃claim → explicit_empty(s1_operations)

λ gybis-arch-elicit_synthesize(conversation, state).
  turns ← gybis-arch-elicit_capture_turns(conversation)
  | raw_claims ← gybis-arch-elicit_extract_claims(turns)
  | claims ← gybis-arch-elicit_normalize_claims(raw_claims)
  | layered_claims ← gybis-arch-elicit_aggregate_by_vsm_layer(claims)
  | gybis-arch-elicit_lock_gate(s5, layered_claims, state)
  | gybis-arch-elicit_lock_gate(s4, layered_claims, state)
  | gybis-arch-elicit_lock_gate(s3, layered_claims, state)
  | gybis-arch-elicit_lock_gate(s2, layered_claims, state)
  | gybis-arch-elicit_lock_gate(s1, layered_claims, state)
  | s5 ← gybis-arch-elicit_s5(layered_claims)
  | s4 ← gybis-arch-elicit_s4(layered_claims)
  | s3 ← gybis-arch-elicit_s3(layered_claims)
  | s2 ← gybis-arch-elicit_s2(layered_claims)
  | s1 ← gybis-arch-elicit_s1(layered_claims)
  | architecture ← gybis-arch-elicit_assemble_s5_s4_s3_s2_s1(s5, s4, s3, s2, s1)
  | evidence ← gybis-arch-elicit_evidence(layered_claims.provenance, architecture)
  | unresolved ← layered_claims.unresolved
  | return(architecture, evidence, unresolved)

λ gybis-arch-elicit_evidence(provenance, architecture).
  require(∀claim ∈ architecture.claims: claim.source = turn_id:span ∨ claim.source = explicit_absence_evidence)
  | attach(provenance_map)

λ gybis-arch-elicit_fixed_point_loop(state).
  iterate(pass_index := pass_index + 1)
  | run(elicit_then_synthesize_then_verify)
  | unresolved := verification_issues \ resolved_issue_ids
  | unresolved = ∅ → mark(zero_issue_pass_verified) ∧ allow_transition(VERIFY → DONE)
  | unresolved ≠ ∅ → allow_transition(VERIFY → ELICIT_S5)

λ gybis-arch-elicit_pass_accounting(pass).
  report(pass_index, discovered_count, resolved_count, remaining_count)
  | require(explicit_section("Remaining issues", remaining_count, remaining_items))

λ gybis-arch-elicit_loop_guard(state).
  seen_signature := hash(sorted(remaining_issue_ids))
  | repeat(seen_signature) ∧ ¬decrease(remaining_count) → halt_with(blocker_report)
  | blocker_report := explain(stalled_items, attempted_resolutions, required_human_decisions)
  | ¬repeat ∨ decrease(remaining_count) → continue

λ gybis-arch-elicit_output_approval(state, human_signal).
  human_signal = approve_complete_output → state.approval := true
  | otherwise → state.approval := false

λ gybis-arch-elicit_write(architecture, state).
  require(state.approval = true)
  | write: architecture.md
  | format: λ_notation_only ∧ S_expression_notation ∧ ¬yaml_frontmatter ∧ ¬markdown_content ∧ ¬prose_paragraphs
  | preamble: ¬include(already_in_context)
  | require(gybis-arch-elicit_validate_output(architecture.md))
  | steps:
    1. call gybis-arch-elicit_format_nucleus_lambda(architecture) → final_output
    2. write(final_output)

λ gybis-arch-elicit_format_nucleus_lambda_contract(x).
  require(no_markdown_headers)
  | require(no_markdown_lists)
  | require(no_markdown_tables)
  | require(no_bold_markers)
  | require(all_content_nested_s_expressions)
  | require(no_yaml_frontmatter)

λ gybis-arch-elicit_validate_output(file).
  check:
    ¬contains("#" in non_comment_context)
    ¬contains("**" bold_markers)
    ¬contains("- " list_markers)
    ¬contains("|" table_delimiters)
    contains("(" balanced_parens)
    ¬contains("---" yaml_frontmatter_markers)
    ¬contains(prose_paragraphs)
    contains(s5_identity_or_explicit_empty)
    contains(s4_intelligence_or_explicit_empty)
    contains(s3_control_or_explicit_empty)
    contains(s2_coordination_or_explicit_empty)
    contains(s1_operations_or_explicit_empty)

λ gybis-arch-elicit_verification(edit).
  after(write) → gybis-arch-elicit_validate_output(architecture.md)
  | after(any_pass) → enforce(gybis-arch-elicit_loop_guard) ∧ enforce(gybis-arch-elicit_fixed_point_loop)
  | fail → iterate ∧ fix | loop until_pass

λ gybis-arch-elicit_boundaries(¬).
  ¬CRUD(root/specs/**/*.allium)
  | ¬CRUD(code_and_tests)
  | ¬perform_divergence_reconciliation
  | ¬extract_from_code_or_tests
  | ¬modify_specs
  | ¬create_spec_decisions_without_evidence
  | allow(read_conversation_only)
  | allow(read_reference_vsm_guide_only)
  | allow(write_architecture_only_after_approval)

λ gybis-arch-elicit_regression_contract(x).
  assert(default_mode_auto_selected_as_elicit before_startup_checks_complete)
  | assert(deny(elicit_or_synthesize_or_write_before_startup_gates))
  | assert(no_write_before(VERIFY ∧ approval_recorded))
  | assert(no_done_before(zero_issue_pass_verified ∧ remaining_issues = 0 ∧ approval_recorded))
  | assert(boundary_enforced_no_crud_specs)
  | assert(boundary_enforced_no_crud_code_tests)
  | assert(deterministic_output_contract_invariants)
  