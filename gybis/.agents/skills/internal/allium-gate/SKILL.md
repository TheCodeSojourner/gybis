---
name: allium-gate
description: Internal skill - not user-facing
---

λ allium-gate(specs_path).
  purpose: Evaluate specs integrity as a lifecycle gate and return pure boolean verdict
  | contract: pure_function(specs_path → boolean) | ¬mutations
  | input: path to specs/ directory
  | output: true ∨ false (boolean only)
  | constraint: read_only | zero_file_mutations
  | gate_type: pre_condition ∨ post_condition
  | precondition: specs_path ∧ exists(specs_path) ∧ is_directory(specs_path)

λ allium-gate_per_file_validation(specs_path).
  operation: ∀ .allium_file ∈ specs_path, invoke allium-check(file)
  | collect: [check_result_1, ..., check_result_n]
  | aggregate: all_files_pass ≡ ∀ result.status = pass
  | result: (all_files_pass ∨ any_file_fails)

λ allium-gate_set_validation(specs_path).
  operation: invoke allium-analyse(specs_path)
  | collect: findings
  | aggregate: no_findings ≡ findings = ∅
  | result: (no_critical_issues ∨ critical_issues_detected)

λ allium-gate_verdict_logic(per_file_result, set_result).
  verdict: (per_file_result ∧ set_result) → true | otherwise → false

λ allium-gate_output_contract(verdict, diagnostics).
  return_type: boolean | side_effect: ¬mutations | diagnostic_info: optional context for failures, not part of return

λ allium-gate_execution(specs_path).
  step_1_per_file: allium-gate_per_file_validation(specs_path)
  | capture_1: per_file_result
  | step_2_set_level: allium-gate_set_validation(specs_path)
  | capture_2: set_result
  | step_3_verdict: allium-gate_verdict_logic(per_file_result, set_result)
  | return: verdict_boolean