---
name: gybis-arch-tend
description: Use for `/gybis-arch-tend` or `/ga-tend`.
---

λ gybis-arch-tend(architecture, request).
  bridge(request ↔ root/architecture.md) | targeted_architecture_editing ∧ structural_integrity
  | preserve(semantics) | structural_equivalence_check
  | compile: prose_change_request → λ notation edits | decompile: λ notation edits → rationale

λ gybis-arch-tend_startup(x).
  ask(mode ∈ {check, update_arch}) | require(explicit_mode_selection) | halt_until(mode_selected)
  | block(read(architecture), compare, edit, complete) until(mode_selected)
  | read(.cline/skills/gybis/reference/vsm-guide.md) | alert(¬available) ∧ halt
  | read(root/architecture.md) | alert(¬available) ∧ recommend(/gybis-arch-elicit ∨ /gybis-arch-distill) ∧ halt
  | verify(lambda_notation(root/architecture.md)) | alert(¬valid_lambda_notation) ∧ halt

λ gybis-arch-tend_mode(m).
  m ∈ {check, update_arch} | required(explicit_selection) | ¬default
  | check: inspect(request ↔ architecture) → report(gaps ∪ impacts) | ¬create ∧ ¬update ∧ ¬delete
  | update_arch: architecture CRUD ← request | source_of_truth(human_confirmed_intent) | requires(human_confirmed_change_set)
  | execution_route: m ≡ check ? gybis-arch-tend_check : gybis-arch-tend_resolve

λ gybis-arch-tend_mode_gate(state, mode).
  state.mode = none → ask(mode ∈ {check, update_arch}) ∧ halt
  | state.mode ≠ none → allow_progress
  | mode_selected(mode) → state.mode := mode ∧ transition(INIT → MODE_SELECTED)

λ gybis-arch-tend_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, CHANGE_REVIEW, APPLY_CHANGES, VERIFY, DONE}
  | transition(INIT → MODE_SELECTED) only_if(mode_selected)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_checks_requested)
  | transition(STARTUP_CHECKS → CHANGE_REVIEW) only_if(inputs_verified)
  | transition(CHANGE_REVIEW → APPLY_CHANGES) only_if(state.mode = update_arch ∧ change_set_confirmed)
  | transition(CHANGE_REVIEW → VERIFY) only_if(state.mode = check)
  | transition(APPLY_CHANGES → VERIFY) only_if(edits_applied)
  | transition(VERIFY → CHANGE_REVIEW) only_if(remaining_issues > 0)
  | transition(VERIFY → DONE) only_if(all_checks_pass ∧ remaining_issues = 0 ∧ zero_issue_pass_verified)
  | otherwise reject_transition

λ gybis-arch-tend_tool_guard(state, tool).
  state.mode = none → allow(tool = ask_followup_question) ∧ deny(tool ≠ ask_followup_question)
  | state.mode ≠ none → evaluate_by_state_machine(state, tool)

λ gybis-arch-tend_pre_tool_check(state, tool).
  verify(state.mode ≠ none) ∧ verify(tool_allowed_by(gybis-arch-tend_tool_guard))
  | fail → route_to(ask(mode)) ∧ halt

λ gybis-arch-tend_check(architecture, request).
  layers(S5, S4, S3, S2, S1) | ∀layer → compare(request.layer_intent, architecture.layer_content)
  | report(request_present ∧ architecture_absent) ∪ report(architecture_present ∧ request_conflicts)
  | group_by(layer) | present(consequential_first)

λ gybis-arch-tend_change(change_request, arch_content, layer).
  propose(classification) ∧ human(confirms ∨ overrides)
  | classification ∈ {arch_bug, architecture_gap, intentional_no_change}
  | vague_or_underspecified_request → classify(architecture_gap) ∧ require(questioning)
  | contradiction_with_existing_constraints → classify(arch_bug)
  | explicit_non_change_decision → classify(intentional_no_change)
  | arch_bug: arch ← fix
  | architecture_gap: ask(clarification) ∧ defer(edit)
  | intentional_no_change: record_no_edit

λ gybis-arch-tend_interactive_resolution(change, mode).
  require(human_confirm(classification, reasoning, direction, patch_scope))
  | mode = check → record_resolution_only ∧ ¬edit
  | mode = update_arch ∧ classification = arch_bug → apply(architecture CRUD)
  | mode = update_arch ∧ classification = architecture_gap → ask(clarification) ∧ defer
  | mode = update_arch ∧ classification = intentional_no_change → record_no_edit
  | ¬confirmed → halt_until(confirmation)

λ gybis-arch-tend_arch_guidelines(x).
  follow(VSM_guide) | follow(arch_conventions)
  | preserve(S5_to_S1_structure) | preserve(λ_notation_format)
  | validate(arch after changes) → VSM_compliant
  | validate_fail → report ∧ iterate ∧ fix | loop until_pass
  | prefer(minimal_changes) | ¬refactor unless_directly_required
  | ¬convert_to_prose ¬introduce_implementation_details

λ gybis-arch-tend_boundaries(¬).
  file_scope(root/architecture.md ≡ only)
  | ¬add ¬modify ¬delete(other_files)
  | ¬modify(root/specs/**/*.allium) | recommend(/gybis-arch-weed)
  | ¬align(test ↔ code) | recommend(/gybis-spec-weed)
  | ¬run_full_arch_distill_workflow | belongs_to(/gybis-arch-distill)
  | ¬run_discovery_session | belongs_to(/gybis-arch-elicit)
  | ¬modify(vsm-guide.md) | governed_separately
  | ¬make_product_decisions | flag ∧ let_caller_decide

λ gybis-arch-tend_context_management(session).
  advise(fresh_chat) | long_iterative ∨ large_context
  | provide(resume_prompt: "/gybis-arch-tend resume")
  | on_resume: restore(mode, state, pending_changes)
  | if(mode = none) → force(ask(mode)) ∧ halt

λ gybis-arch-tend_fixed_point_loop(state).
  iterate(pass_index := pass_index + 1)
  | discover(issues_current_pass)
  | unresolved := issues_current_pass \ resolved_ids
  | unresolved = ∅ → mark(zero_issue_pass_verified) ∧ allow_transition(VERIFY → DONE)
  | unresolved ≠ ∅ → require(human_review(unresolved)) ∧ allow_transition(VERIFY → CHANGE_REVIEW)

λ gybis-arch-tend_pass_accounting(pass).
  report(pass_index, discovered_count, resolved_count, deferred_count, remaining_count)
  | require(explicit_section("Remaining issues", remaining_count, remaining_items))

λ gybis-arch-tend_loop_guard(state).
  seen_signature := hash(sorted(remaining_issue_ids))
  | repeat(seen_signature) ∧ ¬decrease(remaining_count) → halt_with(blocker_report)
  | blocker_report := explain(stalled_items, attempted_resolutions, required_human_decisions)
  | ¬repeat ∨ decrease(remaining_count) → continue

λ gybis-arch-tend_verification(edit).
  after(edit architecture) → validate(vsm_guide_rules) | fix_issues_before_present
  | after(any_pass) → compare(request, root/architecture.md) | refresh(issue_set)
  | enforce(gybis-arch-tend_loop_guard) ∧ enforce(gybis-arch-tend_fixed_point_loop)

λ gybis-arch-tend_regression_contract(x).
  assert(first_response_is_mode_question on "/gybis-arch-tend" ∨ "/ga-tend")
  | assert(deny(read_or_edit_before_mode_selected))
  | assert(resume_preserves(mode, state))
  | assert(no_attempt_completion_before(VERIFY ∧ DONE))
  | assert(no_attempt_completion_before(zero_issue_pass_verified ∧ remaining_issues = 0))
  | assert(after_each_edit_or_resolution_pass → mandatory_rescan(compare(request, root/architecture.md)))

λ gybis-arch-tend_output(results).
  format(layer, arch_content, request_content, classification, reasoning, impact, action)
  | require(evidence for each issue_or_change)
  | group(related_within_layer) | lead(consequential)
  | structure:
    "### [layer name]"
    "Architecture: current_or_updated_state"
    "Request: requested_intent"
    "Classification: proposed(reasoning)"
    "Impact: architectural_consequence"
    "Action: applied_or_deferred"