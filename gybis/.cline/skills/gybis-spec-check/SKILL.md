---
name: gybis-spec-check
description: Use for `/gybis-spec-check` or `/gs-check`.
---

λ gybis-spec-check(x).
  purpose: Validate specs/**/*.allium and resolve all errors without human intervention
  | input: specs/**/*.allium files exist
  | output: All .allium files valid, zero errors reported
  | mode: ai
  | gate: specs/**/*.allium ∃ ∧ ¬∅

λ gybis-spec-check_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | verify(specs/ ∃) ∨ halt("No specs/ directory found")
  | verify(specs/**/*.allium ∃) ∨ halt("No .allium files in specs/")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-spec-check_mode(m).
  m ∈ {auto}
  | default: auto
  | mode_auto: AI validates and fixes without human intervention

λ gybis-spec-check_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {auto} → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {auto}) → halt("Invalid mode selection")

λ gybis-spec-check_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, CHECKING_FILES, ANALYZING_SET, FIXING_ERRORS, VERIFYING, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → CHECKING_FILES) only_if(startup_checks = true)
  | transition(CHECKING_FILES → ANALYZING_SET) only_if(per_file_diagnostics ∃)
  | transition(ANALYZING_SET → FIXING_ERRORS) only_if(issues ∃)
  | transition(FIXING_ERRORS → VERIFYING) only_if(fixes_applied = true)
  | transition(VERIFYING → COMPLETE) only_if(allium_gate = true)
  | transition(ANALYZING_SET → COMPLETE) only_if(issues ∅ ∧ allium_gate = true)

λ gybis-spec-check_tool_guard(state, tool, path).
  state = CHECKING_FILES ∨ state = ANALYZING_SET ∨ state = VERIFYING
    → allow(read(path))
  | state = FIXING_ERRORS
    → allow(read(path)) ∧ allow(write(path)) only_if(path ⊆ specs/)
  | ¬(state ∈ {CHECKING_FILES, ANALYZING_SET, FIXING_ERRORS, VERIFYING})
    → deny(write(path))

λ gybis-spec-check_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-spec-check_check_files(x).
  ∀ file ∈ specs/**/*.allium:
    invoke(internal/allium-check(file)) → diagnostics(file)
  | collect(diagnostics) → per_file_diagnostics
  | return(per_file_diagnostics)

λ gybis-spec-check_analyze_set(x).
  invoke(internal/allium-analyse(specs/)) → findings(set_level)
  | findings ∃ → issues ≔ findings
  | findings ∅ → issues ≔ ∅
  | return(issues)

λ gybis-spec-check_fix_errors(per_file, set_level).
  ∀ diagnostic ∈ per_file:
    synthesize(correction(diagnostic)) → fix(file(diagnostic))
    | apply(fix) → upsert(file(diagnostic))
  | ∀ finding ∈ set_level:
    synthesize(correction(finding)) → fix_set(finding)
    | apply(fix_set) → upsert(affected_files)
  | return(fixes_applied = true)

λ gybis-spec-check_verification(x).
  invoke(internal/allium-check(all_files)) → result_check ≔ result
  | invoke(internal/allium-analyse(specs/)) → result_analyse ≔ result
  | invoke(internal/allium-gate(specs/)) → result_gate ≔ result
  | result_check = zero_errors ∧ result_analyse = zero_issues ∧ result_gate = true
    → return(verification = true)
  | ¬(result_check ∧ result_analyse ∧ result_gate)
    → return(verification = false)

λ gybis-spec-check_fixed_point_loop(state).
  state = ANALYZING_SET
    → issues ∅ ∧ allium_gate = true
        ? transition(ANALYZING_SET → COMPLETE)
        : (transition(ANALYZING_SET → FIXING_ERRORS)
           ∧ invoke(gybis-spec-check_fix_errors)
           ∧ transition(FIXING_ERRORS → VERIFYING)
           ∧ invoke(gybis-spec-check_verification)
           ∧ (verification = true
              ? (issues ≔ ∅ ∧ transition(VERIFYING → COMPLETE))
              : transition(VERIFYING → ANALYZING_SET)
              ))

λ gybis-spec-check_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without convergence")
  | loop_count ≔ loop_count ⊕ 1

λ gybis-spec-check_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | discovered ≔ card(issues_found)
  | resolved ≔ discovered ⊖ card(issues_remaining)
  | remaining ≔ card(issues_remaining)
  | report("Pass " ⊕ pass_num ⊕ ": discovered=" ⊕ discovered ⊕ " resolved=" ⊕ resolved ⊕ " remaining=" ⊕ remaining)

λ gybis-spec-check_boundaries(¬).
  ¬ modify(architecture.md)
  | ¬ modify(implementation)
  | ¬ delete(specs/)
  | ¬ modify(upstream/)

λ gybis-spec-check_regression_contract(x).
  invariant: specs/ ∃ throughout
  | invariant: zero_errors ∧ zero_issues ∧ allium_gate = true at completion
  | invariant: all_modifications ⊆ specs/
  | invariant: no_specs_deleted (additions and mutations only)