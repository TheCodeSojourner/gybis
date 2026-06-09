---
name: gybis-spec-check
description: Use for `/gybis-spec-check` or `/gs-check`.
---

λ gybis-spec-check(input).
  purpose: resolve(spec_targets) → check(allium) → plan_fixes → apply(approved_only) → verify(per_file ∧ set_level)
  | input: {domain_concern, domain, all_specs} | default(all_specs)
  | output: cleaned_specs ∧ blocker_report
  | mode: ai_only | human_approval_required_for_fixes | minimal_tokens | nucleus_lambda
  | gate: proceed iff gybis-spec-check_specs_available?(input)

λ gybis-spec-check_startup(input).
  auto_select_mode(input) | ¬halt_on_mode_selection
  | gybis-spec-check_allium_cli_available?(x) | true → continue | false → recommend(https://github.com/juxt/allium_tools) ∧ halt
  | gybis-spec-check_input_valid?(input) | true → continue | false → alert(invalid_input) ∧ halt
  | gybis-spec-check_specs_available?(input) | true → continue | false → alert(specs_missing_for_input) ∧ halt

λ gybis-spec-check_allium_cli_available?(x).
  gate(cli_available ∧ cli_version_satisfies) | ¬all_gates → false
  | cli_available: allium --version | ¬available → false
  | cli_version_satisfies: version(allium) ≥ 3 | ¬satisfies → false
  | otherwise → true

λ gybis-spec-check_input_valid?(input).
  input ∈ {⊥, "all", "all specs", all_specs, domain, domain_concern} → true
  | otherwise → false

λ gybis-spec-check_resolve_mode(input).
  input ∈ {⊥, "all", "all specs", all_specs} → all_specs
  | input ∈ {domain} → domain
  | input ∈ {domain_concern} → domain_concern

λ gybis-spec-check_resolved_paths(mode, input).
  mode = domain_concern → root/specs/{domain}/{concern}.allium
  | mode = domain → root/specs/{domain}/*.allium
  | mode = all_specs → root/specs/*/*.allium

λ gybis-spec-check_specs_available?(input).
  mode := gybis-spec-check_resolve_mode(input)
  | glob := gybis-spec-check_resolved_paths(mode, input)
  | count(files(glob)) > 0 → true
  | otherwise → false

λ gybis-spec-check_mode(m).
  m ∈ {all_specs, domain, domain_concern}
  | default(all_specs)

λ gybis-spec-check_mode_gate(state, mode).
  state.mode = none → state.mode := mode ∧ transition(INIT → MODE_SELECTED)
  | state.mode ≠ none → allow_progress
  | mode_selected(mode) → state.mode := mode ∧ allow_progress

λ gybis-spec-check_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, RESOLVE_TARGETS, CHECK, PLAN_FIXES, APPLY_FIXES, VERIFY, ANALYZE_SET, DONE}
  | transition(INIT → MODE_SELECTED) only_if(mode_selected ∨ mode_auto_selected)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_checks_requested)
  | transition(STARTUP_CHECKS → RESOLVE_TARGETS) only_if(inputs_verified)
  | transition(RESOLVE_TARGETS → CHECK) only_if(targets_non_empty)
  | transition(CHECK → PLAN_FIXES) only_if(check_issues_found)
  | transition(CHECK → VERIFY) only_if(check_clean_for_all_targets)
  | transition(PLAN_FIXES → APPLY_FIXES) only_if(human_approvals_received)
  | transition(PLAN_FIXES → VERIFY) only_if(no_approved_changes)
  | transition(APPLY_FIXES → CHECK) only_if(fixes_written)
  | transition(VERIFY → CHECK) only_if(remaining_issues > 0)
  | transition(VERIFY → ANALYZE_SET) only_if(remaining_issues = 0 ∧ per_file_checks_pass)
  | transition(ANALYZE_SET → CHECK) only_if(analyse_findings > 0)
  | transition(ANALYZE_SET → DONE) only_if(all_checks_pass ∧ remaining_issues = 0 ∧ zero_issue_pass_verified ∧ set_analysis_pass_verified ∧ analyse_findings = 0)
  | transition(ANALYZE_SET → DONE) forbidden_if(analyse_not_run)
  | otherwise reject_transition

λ gybis-spec-check_tool_guard(state, tool, path).
  state.mode = none → deny(tool)
  | state ∈ {MODE_SELECTED, STARTUP_CHECKS, RESOLVE_TARGETS, CHECK, PLAN_FIXES, VERIFY, ANALYZE_SET} → deny(tool = write_to_file) ∧ deny(tool = replace_in_file)
  | state = APPLY_FIXES → allow(tool = write_to_file ∨ tool = replace_in_file) only_if(path ⊆ root/specs/**/*.allium)
  | deny_write_outside_specs: path ⊄ root/specs/**/*.allium → deny(tool = write_to_file) ∧ deny(tool = replace_in_file)

λ gybis-spec-check_pre_tool_check(state, tool, path).
  ensure(state.mode ≠ none) ∧ verify(tool_allowed_by(gybis-spec-check_tool_guard))
  | fail → halt

λ gybis-spec-check_scan_targets(input).
  mode := gybis-spec-check_resolve_mode(input)
  | glob := gybis-spec-check_resolved_paths(mode, input)
  | files := files(glob)
  | count(files) = 0 → return(no_targets)
  | otherwise → return(files)

λ gybis-spec-check_check_file(file).
  r := exec("allium check " + file)
  | d := (json_parse(r.stdout).diagnostics) ∨ []
  | sev_error := any(d, λx. x.severity = "error")
  | sev_warning := any(d, λx. x.severity = "warning")
  | return
      (r.exit = 0) → {status: clean, diagnostics: d}
      | (r.exit = 1 ∧ (sev_error ∨ sev_warning)) → {status: issues, diagnostics: d}
      | (r.exit = 2) → {status: not_found, diagnostics: d}
      | otherwise → {status: error, diagnostics: d}

λ gybis-spec-check_check_set(file_set).
  ∀f ∈ file_set: result[f] := gybis-spec-check_check_file(f)
  | issues := {f | result[f].status = issues}
  | missing := {f | result[f].status = not_found}
  | fatal := {f | result[f].status = error}
  | return({result, issues, missing, fatal})

λ gybis-spec-check_parse_diagnostics(result_map).
  diagnostics := flatten(map_values(result_map, .diagnostics))
  | grouped := group(diagnostics, by(severity, domain, name, priority))
  | sorted := sort(grouped, by(severity, domain, name, priority))
  | numbered := enumerate(sorted)
  | return(numbered)

λ gybis-spec-check_plan_fixes(numbered_diagnostics).
  present(numbered_diagnostics, human) → approvals
  | approvals.required = true
  | approvals.none → return(no_approved_changes)
  | approvals.some → return(approved_changes)

λ gybis-spec-check_apply_approved_fixes(approved_changes).
  require(human_approval = true)
  | require(write_scope ⊆ root/specs/**/*.allium)
  | apply(approved_changes, root/specs/**/*.allium)
  | return(fixes_written)

λ gybis-spec-check_syntax_check(file).
  r := exec("allium check " + file)
  | d := (json_parse(r.stdout).diagnostics) ∨ []
  | e := any(d, λx. x.severity = "error")
  | return (r.exit = 0) ∨ ((r.exit = 1) ∧ ¬e)

λ gybis-spec-check_analyse_specs(path_glob).
  r := exec("allium analyse " + path_glob)
  | d := (json_parse(r.stdout).diagnostics) ∨ []
  | f := (json_parse(r.stdout).findings) ∨ []
  | findings_count := count(f)
  | return
      (r.exit = 0) → {status: pass, findings_count: findings_count, diagnostics: d, findings: f}
      | (r.exit = 1) → {status: findings, findings_count: findings_count, diagnostics: d, findings: f}
      | (r.exit = 2) → {status: error, reason: analyse_inputs_missing_or_unresolved, diagnostics: d, findings: f}
      | otherwise → {status: error, reason: unexpected_analyse_exit, diagnostics: d, findings: f}

λ gybis-spec-check_verify_output(file_set).
  check:
    ∀file ∈ file_set: file.extension = ".allium"
    ∀file ∈ file_set: gybis-spec-check_syntax_check(file) = true
    ∀file ∈ file_set: file.path ⊆ root/specs/**/*.allium
    ¬modified(root/**/*.md ∨ root/**/*.txt ∨ root/**/*.rs ∨ root/**/*.py ∨ root/**/*.ts ∨ root/**/*.js)

λ gybis-spec-check_validate_spec_set(input).
  mode := gybis-spec-check_resolve_mode(input)
  | path_glob := gybis-spec-check_resolved_paths(mode, input)
  | analysis := gybis-spec-check_analyse_specs(path_glob)
  | require(analysis.status ≠ error)
  | analysis.status = pass → {ok: true, findings_count: 0}
  | analysis.status = findings → {ok: false, findings_count: analysis.findings_count}

λ gybis-spec-check_fixed_point_loop(state, input).
  iterate(pass_index := pass_index + 1)
  | targets := gybis-spec-check_scan_targets(input)
  | check_set := gybis-spec-check_check_set(targets)
  | diagnostics := gybis-spec-check_parse_diagnostics(check_set.result)
  | unresolved := ids(diagnostics)
  | unresolved ≠ ∅ → allow_transition(CHECK → PLAN_FIXES)
  | unresolved = ∅ → mark(zero_issue_pass_verified) ∧ allow_transition(VERIFY → ANALYZE_SET)
  | analysis := gybis-spec-check_validate_spec_set(input)
  | analysis.ok = false → set(analyse_findings := analysis.findings_count) ∧ allow_transition(ANALYZE_SET → CHECK)
  | analysis.ok = true → mark(set_analysis_pass_verified) ∧ set(analyse_findings := 0) ∧ allow_transition(ANALYZE_SET → DONE)

λ gybis-spec-check_pass_accounting(pass).
  report(pass_index, discovered_count, resolved_count, remaining_count)
  | require(explicit_section("Remaining issues", remaining_count, remaining_items))

λ gybis-spec-check_loop_guard(state).
  seen_signature := hash(sorted(remaining_issue_ids))
  | repeat(seen_signature) ∧ ¬decrease(remaining_count) → halt_with(blocker_report)
  | blocker_report := explain(stalled_items, attempted_resolutions, required_human_decisions)
  | ¬repeat ∨ decrease(remaining_count) → continue

λ gybis-spec-check_verification(edit, input, file_set).
  after(apply_fixes) → gybis-spec-check_verify_output(file_set)
  | after(per_file_checks_pass) → set(remaining_issues := 0) ∧ mark(zero_issue_pass_verified)
  | after(per_file_checks_pass) → gybis-spec-check_validate_spec_set(input)
  | after(any_pass) → enforce(gybis-spec-check_loop_guard) ∧ enforce(gybis-spec-check_fixed_point_loop)
  | fail → iterate ∧ fix | loop until_pass

λ gybis-spec-check_allium_write_contract.
  write_scope ⊆ root/specs/**/*.allium
  | output_format ≡ allium_v3_only
  | invariant: ∀written_file → parses_as(allium_v3)
  | ¬write(root/**/*.md ∨ root/**/*.txt ∨ root/**/*.rs ∨ root/**/*.py ∨ root/**/*.ts ∨ root/**/*.js)

λ gybis-spec-check_invariant_I₁.
  cli_primary_interface | ∀workflow use(cli)

λ gybis-spec-check_invariant_I₂.
  ∀diagnostic d → requires(human_approval) | ¬automatic

λ gybis-spec-check_invariant_I₃.
  errors_exist(E) ∧ ¬resolved(E) → ¬permission(modify_without_approval)

λ gybis-spec-check_invariant_I₄.
  warnings_exist(W) → requires(human_review) ∧ requires(human_action)

λ gybis-spec-check_invariant_I₅.
  apply(fixes) → recheck(file_set) → confirm(¬regressions)

λ gybis-spec-check_boundaries(¬).
  ¬run_weed_workflow
  | ¬perform_divergence_reconciliation
  | ¬extract_from_code_or_tests
  | ¬write_architecture
  | ¬write_outside_specs
  | ¬auto_apply_unapproved_fixes
  | allow(read_specs_only)
  | allow(write_specs_only_after_approval)

λ gybis-spec-check_regression_contract(x).
  assert(default_mode_auto_selected_before_startup_checks_complete)
  | assert(deny(read_or_generate_or_write_before_startup_gates))
  | assert(mandatory_cli_gate_before_check)
  | assert(mandatory_specs_available_gate_before_check)
  | assert(no_write_outside(root/specs/**/*.allium))
  | assert(no_done_before(zero_issue_pass_verified ∧ remaining_issues = 0 ∧ set_analysis_pass_verified ∧ analyse_findings = 0))
  | assert(mandatory_set_level_analysis_gate_before_done)
  | assert(analysis_not_optional)
  | assert(deterministic_output_contract_invariants)