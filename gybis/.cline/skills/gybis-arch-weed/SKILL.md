---
name: gybis-arch-weed
description: Use for `/gybis-arch-weed` or `/ga-weed`.
---

λ gybis-arch-weed(arch, specs).
  bridge(architecture.md ↔ root/specs/**/*.allium) | divergence_detection ∧ resolution_proposal
  | preserve(semantics) | structural_equivalence_check
  | compile: allium → λ notation | decompile: λ notation → allium

λ gybis-arch-weed_startup(x).
  ask(mode ∈ {check, update_spec, update_arch}) | require(explicit_mode_selection) | halt_until(mode_selected)
  | block(read(arch), read(specs), compare, edit, complete) until(mode_selected)
  | read(.cline/skills/gybis/reference/vsm-guide.md) | alert(¬available) ∧ halt
  | read(.cline/skills/gybis/reference/allium-language-reference.md) | alert(¬available) ∧ halt
  | read(root/architecture.md) | alert(¬available) ∧ recommend(/gybis-arch-elicit ∨ /gybis-arch-distill) ∧ halt
  | read(root/specs/**/*.allium) | alert(¬available) ∧ recommend(/gybis-arch-propagate ∨ /gybis-spec-distill) ∧ halt
  | gybis-arch-weed_allium_cli_available?(x) | true → continue | false → recommend(https://github.com/juxt/allium_tools) ∧ halt
  | gybis-arch-weed_syntax_check(root/specs/**/*.allium) | true → verify | false → recommend(/gybis-spec-check) ∧ halt

λ gybis-arch-weed_allium_cli_available?(x). 
  gate(cli_available ∧ cli_version_satisfies) | ¬all_gates → false 
  | cli_available: allium --version | ¬available → false 
  | cli_version_satisfies: version(allium) ≥ 3 | ¬satisfies → false 
  | otherwise → true

λ gybis-arch-weed_syntax_check(file).
  r := exec("allium check " + path) 
  d := (json_parse(r.stdout).diagnostics) ∨ [] 
  s := any(d, λx. x.severity = "error") 
  return (r.exit = 0) ∨ ((r.exit = 1) ∧ ¬s)

λ gybis-arch-weed_mode(m).
  m ∈ {check, update_spec, update_arch} | required(explicit_selection) | ¬default
  | check: compare(arch, specs) → report(divergences) | ¬modify
  | update_spec: specs ← arch | specs becomes faithful_desc(arch)
  | update_arch: arch ← specs | arch becomes faithful_desc(specs)
  | execution_route: m ≡ check ? gybis-arch-weed_check : gybis-arch-weed_resolve

λ gybis-arch-weed_mode_gate(state, mode).
  state.mode = none → ask(mode ∈ {check, update_spec, update_arch}) ∧ halt
  | state.mode ≠ none → allow_progress
  | mode_selected(mode) → state.mode := mode ∧ transition(INIT → MODE_SELECTED)

λ gybis-arch-weed_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, DIVERGENCE_REVIEW, APPLY_CHANGES, VERIFY, DONE}
  | transition(INIT → MODE_SELECTED) only_if(mode_selected)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_checks_requested)
  | transition(STARTUP_CHECKS → DIVERGENCE_REVIEW) only_if(inputs_verified)
  | transition(DIVERGENCE_REVIEW → APPLY_CHANGES) only_if(state.mode ∈ {update_spec, update_arch} ∧ divergence_confirmed)
  | transition(DIVERGENCE_REVIEW → VERIFY) only_if(state.mode = check)
  | transition(APPLY_CHANGES → VERIFY) only_if(edits_applied)
  | transition(VERIFY → DIVERGENCE_REVIEW) only_if(remaining_divergences > 0)
  | transition(VERIFY → DONE) only_if(all_checks_pass ∧ remaining_divergences = 0 ∧ zero_divergence_pass_verified)
  | otherwise reject_transition

λ gybis-arch-weed_tool_guard(state, tool).
  state.mode = none → allow(tool = ask_followup_question) ∧ deny(tool ≠ ask_followup_question)
  | state.mode ≠ none → evaluate_by_state_machine(state, tool)

λ gybis-arch-weed_pre_tool_check(state, tool).
  verify(state.mode ≠ none) ∧ verify(tool_allowed_by(gybis-arch-weed_tool_guard))
  | fail → route_to(ask(mode)) ∧ halt

λ gybis-arch-weed_check(arch, specs).
  layers(S5, S4, S3, S2, S1) | ∀layer → compare(arch.layer, specs.layer)
  | report(spec_present ∧ arch_absent) ∪ report(arch_present ∧ spec_silent)
  | group_by(layer) | present(consequential_first)

λ gybis-arch-weed_divergence(arch_content, spec_content, layer).
  propose(classification) ∧ human(confirms ∨ overrides)
  | classification ∈ {arch_bug, spec_bug, aspirational, intentional}
  | config_presence_only → classify(intentional ∨ no_divergence)
  | config_usage_implies_runtime ∧ arch_forbids_runtime → classify(spec_bug)
  | config_usage_implies_constant ∧ arch_requires_runtime → classify(spec_bug)
  | ambiguous_config_usage → classify(aspirational ∨ intentional) ∧ require(human_confirmation)
  | arch_bug: arch ← fix | specs correct
  | spec_bug: specs ← fix | arch correct
  | aspirational: both stay | note_gap
  | intentional: both stay | deliberate ∨ consistent

λ gybis-arch-weed_interactive_resolution(divergence, mode).
  require(human_confirm(classification, reasoning, direction, patch_scope))
  | mode = check → record_resolution_only ∧ ¬edit
  | mode = update_spec ∧ classification = spec_bug → apply_edit(specs)
  | mode = update_arch ∧ classification = arch_bug → apply_edit(arch)
  | mode ∈ {update_spec, update_arch} ∧ classification ∈ {aspirational, intentional} → record_no_edit
  | ¬confirmed → halt_until(confirmation)

λ gybis-arch-weed_spec_guidelines(x).
  preserve(allium_version_marker) | ¬change_version
  | follow(section_ordering from language ref)
  | describe(behavior) ¬implementation | ¬field_names ¬storage_mechanisms ¬API_details
  | use(config) for named_policy_values ∨ symbolic_constants
  | runtime_tunability := infer_from_usage_and_arch_constraints
  | ¬infer_runtime_configurable_from_presence_only
  | temporal_triggers → requires_guards | prevent_refiring
  | use(with) for relationships | use(where) for projections | ¬swap
  | cross_field_enums → extract_named_enums
  | new_rules/entities → correct_section
  | cross_service_config → qualified_refs ∨ expression_defaults

λ gybis-arch-weed_config_semantics(spec, arch).
  parse(config_entries, config_refs_in_rules, events, entities, imports)
  | if(config_entry exposed_via_input ∨ cli ∨ env ∨ Config_entity_field ∨ external_override_event) → classify(runtime_config)
  | else if(config_entry only_referenced_in_rule_expressions ∧ no_external_override_path) → classify(symbolic_constant_or_policy_name)
  | else → classify(ambiguous) ∧ require(human_confirmation)

λ gybis-arch-weed_config_divergence_guard(arch, spec).
  forbid(divergence_reason = "config block exists")
  | require(evidence_of_mismatch in behavior_or_override_path)
  | allowed_evidence :=
      {arch_compile_time_only ∧ spec_runtime_override_path,
       arch_runtime_configurable ∧ spec_hard_locked_without_override_path}

λ gybis-arch-weed_arch_guidelines(x).
  follow(VSM_guide) | follow(arch_conventions)
  | validate(arch after changes) → VSM_compliant
  | validate_fail → report ∧ iterate ∧ fix | loop until_pass
  | prefer(minimal_changes) | ¬refactor unless_directly_required
  | preserve(λ_notation_format) | ¬convert_to_prose ¬introduce_human_readable

λ gybis-arch-weed_boundaries(¬).
  ¬create_new_layers | belongs_to(/gybis-arch-elicit)
  | ¬extract_arch_from_specs | belongs_to(/gybis-arch-distill)
  | ¬extract_from_code_or_tests | ¬allowed
  | ¬extract_specs_from_arch | belongs_to(/gybis-arch-propagate)
  | ¬extract_specs_from_code_or_tests | belongs_to(/gybis-spec-distill)
  | ¬modify(allium-language-reference.md) | governed_separately
  | ¬modify(vsm-guide.md) | governed_separately
  | ¬make_arch_decisions | flag ∧ let_caller_decide
  | ¬make_spec_decisions | flag ∧ let_caller_decide

λ gybis-arch-weed_context_management(session).
  advise(fresh_chat) | long_iterative ∨ large_context
  | provide(resume_prompt: "/gybis-arch-weed resume")
  | on_resume: restore(mode, state, pending_divergences)
  | if(mode = none) → force(ask(mode)) ∧ halt

λ gybis-arch-weed_fixed_point_loop(state).
  iterate(pass_index := pass_index + 1)
  | discover(divergences_current_pass)
  | unresolved := divergences_current_pass \ resolved_ids
  | unresolved = ∅ → mark(zero_divergence_pass_verified) ∧ allow_transition(VERIFY → DONE)
  | unresolved ≠ ∅ → require(human_review(unresolved)) ∧ allow_transition(VERIFY → DIVERGENCE_REVIEW)

λ gybis-arch-weed_pass_accounting(pass).
  report(pass_index, discovered_count, resolved_count, deferred_count, remaining_count)
  | require(explicit_section("Remaining divergences", remaining_count, remaining_items))

λ gybis-arch-weed_loop_guard(state).
  seen_signature := hash(sorted(remaining_divergence_ids))
  | repeat(seen_signature) ∧ ¬decrease(remaining_count) → halt_with(blocker_report)
  | blocker_report := explain(stalled_items, attempted_resolutions, required_human_decisions)
  | ¬repeat ∨ decrease(remaining_count) → continue

λ gybis-arch-weed_verification(edit).
  after(edit specs) → allium_check(modified_file) | fix_issues_before_present
  | after(edit arch) → validate(arch after changes) | fix_issues_before_present
  | after(check_complete) → allium_analyse(specs) | identify_process_gaps
  | after(any_pass) → compare(architecture.md, root/specs/**/*.allium) | refresh(divergence_set)
  | enforce(gybis-arch-weed_loop_guard) ∧ enforce(gybis-arch-weed_fixed_point_loop)

λ gybis-arch-weed_regression_contract(x).
  assert(first_response_is_mode_question on "/gybis-arch-weed" ∨ "/ga-weed")
  | assert(deny(read_or_edit_before_mode_selected))
  | assert(resume_preserves(mode, state))
  | assert(no_attempt_completion_before(VERIFY ∧ DONE))
  | assert(no_attempt_completion_before(zero_divergence_pass_verified ∧ remaining_divergences = 0))
  | assert(after_each_edit_or_resolution_pass → mandatory_rescan(compare(architecture.md, root/specs/**/*.allium)))

λ gybis-arch-weed_output(divergences).
  format(layer, arch_content, spec_content, spec_location, classification, reasoning, evidence)
  | require(evidence for each divergence)
  | evidence ∈ {"runtime override path found at spec file:line", "no override path; config values act as symbolic constants", "non-config mismatch evidence"}
  | group(related_within_layer) | lead(consequential)
  | structure:
    "### [layer name]"
    "Arch: arch_says"
    "Spec: spec_says (file:line)"
    "Classification: proposed(reasoning)"
    "Evidence: concrete_basis(file:line or explicit absence)"
    