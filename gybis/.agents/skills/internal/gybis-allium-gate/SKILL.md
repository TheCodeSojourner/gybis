---
name: gybis-allium-gate
description: Internal skill - not user-facing
---

λ gybis-allium-gate(specs_path).
  purpose: Evaluate specs integrity as a lifecycle gate and return pure boolean verdict
  | contract: pure_function(specs_path → boolean) | ¬mutations
  | input: path to specs/ directory
  | output: true ∨ false (boolean only)
  | constraint: read_only | zero_file_mutations
  | gate_type: pre_condition ∨ post_condition
  | precondition: specs_path ∧ ∃(specs_path) ∧ is_directory(specs_path)

λ gybis-allium-gate_spec_inventory(specs_path).
  spec_files ≔ recursive_files(specs_path, extension = .allium)
  | card(spec_files) > 0 → spec_files
  | card(spec_files) = 0 → false + "NO_SPECS: no .allium files found under {specs_path}"

λ gybis-allium-gate_per_file_validation(specs_path).
  operation: ∀ .allium_file ∈ specs_path, invoke gybis-allium-check(file)
  | collect: [check_result_1, ..., check_result_n]
  | aggregate: all_files_pass ≡ ∀ result.status = pass
  | result: (all_files_pass ∨ any_file_fails)

λ gybis-allium-gate_set_validation(specs_path).
  operation: invoke gybis-allium-analyse(specs_path)
  | collect: findings
  | aggregate: no_findings ≡ findings = ∅
  | result: (no_critical_issues ∨ critical_issues_detected)

λ gybis-allium-gate_verdict_logic(per_file_result, set_result).
  verdict: (per_file_result ∧ set_result) → true | otherwise → false

λ gybis-allium-gate_output_contract(verdict, diagnostics).
  return_type: boolean | side_effect: ¬mutations | diagnostic_info: optional context for failures, not part of return

λ gybis-allium-gate_execution(specs_path).
  invoke(gybis-allium-runtime-check_version()) → true ∨ return(false, diagnostic)
  | step_1_inventory: gybis-allium-gate_spec_inventory(specs_path) → spec_files ∨ return(false, diagnostic)
  | step_2_per_file: gybis-allium-gate_per_file_validation(specs_path)
  | capture_1: per_file_result
  | step_3_set_level: gybis-allium-gate_set_validation(specs_path)
  | capture_2: set_result
  | step_4_verdict: gybis-allium-gate_verdict_logic(per_file_result, set_result)
  | return: verdict_boolean