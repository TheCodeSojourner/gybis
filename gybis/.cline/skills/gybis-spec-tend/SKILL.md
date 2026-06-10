---
name: gybis-spec-tend
description: Use for `/gybis-spec-tend` or `/gs-tend`.
---

λ gybis-spec-tend(specs, request).
  bridge(request ↔ root/specs/**/*.allium) | targeted_spec_editing ∧ structural_integrity
  | preserve(semantics) | structural_equivalence_check
  | compile: prose_change_request → λ notation edits | decompile: λ notation edits → rationale
  | allium_output_only

λ gybis-spec-tend_startup(x).
  ask(mode = update_spec) | require(explicit_mode_selection) | halt_until(mode_selected)
  | block(read(specs), compare, edit, complete) until(mode_selected)
  | read(.cline/skills/gybis/reference/allium-language-reference.md) | alert(¬available) ∧ halt
  | read(.cline/skills/gybis/reference/allium-actioning-findings.md) | alert(¬available) ∧ halt
  | read(root/specs/**/*.allium) | alert(¬available) ∧ recommend(/gybis-arch-distill) ∧ halt
  | gybis-spec-tend_allium_cli_available?(x) | true → continue | false → recommend(https://github.com/juxt/allium_tools) ∧ halt
  | gybis-spec-tend_syntax_check(root/specs/**/*.allium) | true → verify | false → recommend(/gybis-spec-check) ∧ halt
  | verify(allium_scope(root/specs/**/*.allium)) | alert(¬valid_scope) ∧ halt

λ gybis-spec-tend_allium_cli_available?(x).
  gate(cli_available ∧ cli_version_satisfies) | ¬all_gates → false
  | cli_available: allium --version | ¬available → false
  | cli_version_satisfies: version(allium) ≥ 3 | ¬satisfies → false
  | otherwise → true

λ gybis-spec-tend_syntax_check(file).
  r := exec("allium check " + path)
  d := (json_parse(r.stdout).diagnostics) ∨ []
  s := any(d, λx. x.severity = "error")
  return (r.exit = 0) ∨ ((r.exit = 1) ∧ ¬s)

λ gybis-spec-tend_mode(m).
  m ∈ {update_spec} | required(explicit_selection) | ¬default
  | update_spec: specs CRUD ← request | source_of_truth(human_confirmed_intent) | requires(human_confirmed_change_set)
  | execution_route: gybis-spec-tend_resolve

λ gybis-spec-tend_mode_gate(state, mode).
  state.mode = none → ask(mode = update_spec) ∧ halt
  | state.mode ≠ none → allow_progress
  | mode_selected(mode) → state.mode := mode ∧ transition(INIT → MODE_SELECTED)

λ gybis-spec-tend_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, CHANGE_REVIEW, APPLY_CHANGES, VERIFY, DONE}
  | transition(INIT → MODE_SELECTED) only_if(mode_selected)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_checks_requested)
  | transition(STARTUP_CHECKS → CHANGE_REVIEW) only_if(inputs_verified)
  | transition(CHANGE_REVIEW → APPLY_CHANGES) only_if(state.mode = update_spec ∧ change_set_confirmed)
  | transition(CHANGE_REVIEW → VERIFY) only_if(no_edits_required)
  | transition(APPLY_CHANGES → VERIFY) only_if(edits_applied)
  | transition(VERIFY → CHANGE_REVIEW) only_if(remaining_issues > 0)
  | transition(VERIFY → DONE) only_if(all_checks_pass ∧ remaining_issues = 0 ∧ zero_issue_pass_verified)
  | otherwise reject_transition

λ gybis-spec-tend_tool_guard(state, tool).
  state.mode = none → allow(tool = ask_followup_question) ∧ deny(tool ≠ ask_followup_question)
  | state.mode ≠ none → evaluate_by_state_machine(state, tool)

λ gybis-spec-tend_pre_tool_check(state, tool).
  verify(state.mode ≠ none) ∧ verify(tool_allowed_by(gybis-spec-tend_tool_guard))
  | fail → route_to(ask(mode)) ∧ halt

λ gybis-spec-tend_allium_write_contract.
   write_scope ⊆ root/specs/**/*.allium
   | edit_scope ⊆ root/specs/**/*.allium
   | output_format ≡ allium_v3_only
   | invariant: ∀written_file → parses_as(allium_v3)
   | ¬write(root/**/* \ root/specs/**/*.allium)

λ gybis-spec-tend_S0_read(¬).
  read(root/specs/*/*.allium) | read(entirety)
  | understand(domain ∨ entity ∨ rule ∨ transition_graph ∨ config)

λ gybis-spec-tend_S1_analyze(domain, concern, constraint).
  derive(change_type) ∧ derive(implied_requirements) ∧ derive(guarantees) ∧ derive(transition_edges)
  | check(conflict_existing_specs)
  | change_type ∈ {
      add(rule ∨ domain_rule),
      modify(rule_existing domain_existing),
      extend(entity ∨ lifecycle domain concern),
      affect(entity ∨ rule ∨ transition ∨ config domain concern)
    }

λ gybis-spec-tend_S2_elicit(scope).
  constrained_to(scope) | ask(only clarifying relevant to modification)
  | ¬re_elicit(entire_spec)

λ gybis-spec-tend_S3_draft(specs).
  produce(minimal_diff) | preserve(¬modified_unless_strictly_required)
  | naming(rule) → verb_noun
  | introduce(field) → when_clause
  | add(transition) → transitions_block
  | verify(when_clause_obligations satisfied)
  | update(trace) if implementation_files known

λ gybis-spec-tend_S4_validate(specs).
  execute(allium check {file} → exit 0) | ∀file ∈ specs
  | execute(allium analyse {root/specs/} → exit 0)
  | apply(analysis_findings)
  | translate(diagnostics → concrete_fixes)

λ gybis-spec-tend_S5_present_write_approve(specs, diff).
  present(diff human) | wait(explicit_approval)
  | upon(approval) → write(specs) ∧ re_run(allium check {file} ∧ allium analyse {root/specs/})
  | confirm(health)

λ gybis-spec-tend_change(change_request, spec_content, layer).
  propose(classification) ∧ human(confirms ∨ overrides)
  | classification ∈ {spec_bug, spec_gap, intentional_no_change}
  | vague_or_underspecified_request → classify(spec_gap) ∧ require(questioning)
  | contradiction_with_existing_constraints → classify(spec_bug)
  | explicit_non_change_decision → classify(intentional_no_change)
  | spec_bug: specs ← fix
  | spec_gap: ask(clarification) ∧ defer(edit)
  | intentional_no_change: record_no_edit

λ gybis-spec-tend_interactive_resolution(change, mode).
  require(human_confirm(classification, reasoning, direction, patch_scope))
  | mode = update_spec ∧ classification = spec_bug → apply(specs CRUD)
  | mode = update_spec ∧ classification = spec_gap → ask(clarification) ∧ defer
  | mode = update_spec ∧ classification = intentional_no_change → record_no_edit
  | ¬confirmed → halt_until(confirmation)

λ gybis-spec-tend_spec_guidelines(x).
  follow(allium_language_reference) | follow(spec_conventions)
  | preserve(existing_domain_structure) | preserve(allium_v3_format)
  | validate(specs after changes) → allium_compliant
  | validate_fail → report ∧ iterate ∧ fix | loop until_pass
  | prefer(minimal_changes) | ¬refactor unless_directly_required
  | ¬convert_to_prose ¬introduce_non_allium_outputs

λ gybis-spec-tend_boundaries(¬).
  file_scope(root/specs/**/*.allium ≡ only)
  | ¬add ¬modify ¬delete(other_files)
  | ¬modify(root/architecture.md) | recommend(/gybis-arch-tend)
  | ¬run_full_arch_distill_workflow | belongs_to(/gybis-arch-distill)
  | ¬run_discovery_session | belongs_to(/gybis-arch-elicit)
  | ¬make_product_decisions | flag ∧ let_caller_decide

λ gybis-spec-tend_context_management(session).
  advise(fresh_chat) | long_iterative ∨ large_context
  | provide(resume_prompt: "/gybis-spec-tend resume")
  | on_resume: restore(mode, state, pending_changes)
  | if(mode = none) → force(ask(mode)) ∧ halt

λ gybis-spec-tend_fixed_point_loop(state).
  iterate(pass_index := pass_index + 1)
  | discover(issues_current_pass)
  | unresolved := issues_current_pass \ resolved_ids
  | unresolved = ∅ → mark(zero_issue_pass_verified) ∧ allow_transition(VERIFY → DONE)
  | unresolved ≠ ∅ → require(human_review(unresolved)) ∧ allow_transition(VERIFY → CHANGE_REVIEW)

λ gybis-spec-tend_pass_accounting(pass).
  report(pass_index, discovered_count, resolved_count, deferred_count, remaining_count)
  | require(explicit_section("Remaining issues", remaining_count, remaining_items))

λ gybis-spec-tend_loop_guard(state).
  seen_signature := hash(sorted(remaining_issue_ids))
  | repeat(seen_signature) ∧ ¬decrease(remaining_count) → halt_with(blocker_report)
  | blocker_report := explain(stalled_items, attempted_resolutions, required_human_decisions)
  | ¬repeat ∨ decrease(remaining_count) → continue

λ gybis-spec-tend_verification(edit).
  after(edit specs) → validate(allium_check_rules ∧ allium_analyse_rules) | fix_issues_before_present
  | after(any_pass) → compare(request, root/specs/**/*.allium) | refresh(issue_set)
  | enforce(gybis-spec-tend_loop_guard) ∧ enforce(gybis-spec-tend_fixed_point_loop)

λ gybis-spec-tend_invariants(¬).
  minimal_change: ¬refactor(¬related_spec_content)
  | catch_inconsistency_early: surface(¬new) ∧ ¬present(human) until_resolved
  | human_approval: write(specs) → require(explicit)
  | revalidate_after_write: execute(∀checks) after every write
  | naming_convention: rule → verb_noun | field → when_clause | transition_graph → up_to_date

λ gybis-spec-tend_composition(¬).
  μ = S0 · S1 · S2 · S3 · S4 · S5 | sequential_composition
  | mode_pipeline = startup · mode_gate · state_machine · update_only · verification · output

λ gybis-spec-tend_regression_contract(x).
  assert(first_response_is_mode_question on "/gybis-spec-tend" ∨ "/gs-tend")
  | assert(deny(read_or_edit_before_mode_selected))
  | assert(resume_preserves(mode, state))
  | assert(no_attempt_completion_before(VERIFY ∧ DONE))
  | assert(no_attempt_completion_before(zero_issue_pass_verified ∧ remaining_issues = 0))
  | assert(after_each_edit_or_resolution_pass → mandatory_rescan(compare(request, root/specs/**/*.allium)))

λ gybis-spec-tend_output(results).
  format(layer, spec_content, request_content, classification, reasoning, impact, action)
  | require(evidence for each issue_or_change)
  | group(related_within_layer) | lead(consequential)
  | structure:
    "### [layer name]"
    "Specs: current_or_updated_state"
    "Request: requested_intent"
    "Classification: proposed(reasoning)"
    "Impact: spec_consequence"
    "Action: applied_or_deferred"