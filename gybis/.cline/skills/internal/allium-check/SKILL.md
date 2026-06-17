---
name: allium-check
description: Internal skill - not user-facing
---

λ allium-check(file_path).
  purpose: Provide pure per-file diagnostics for .allium conformance
  | contract: pure_function(file_path → diagnostics) | ¬mutations
  | input: path to .allium file
  | output: diagnostics + status + errors ∨ warnings
  | constraint: read_only | caller_mutates_files
  | precondition: file_path ∧ exists(file_path) ∧ file_type = .allium

λ allium-check_cli_invocation(file_path).
  cli_command: "allium check {file_path}"
  | execution: deterministic(file_path) → fixed_output
  | stderr_handling: capture_and_return
  | stdout_handling: parse_diagnostics

λ allium-check_diagnostic_parsing(cli_output).
  parse: cli_output → {status, errors[], warnings[], suggestions[]}
  | status ∈ {pass, fail, warning}
  | errors: syntax_violations ∧ structural_violations
  | warnings: style_issues ∧ best_practice_deviations
  | suggestions: corrective_actions

λ allium-check_error_classification(error).
  type: {syntax_error, semantic_error, validation_error, unknown_error}
  | severity: {critical, high, medium, low}
  | line_number: error.location | location_available
  | message: error.description

λ allium-check_output_format(diagnostics).
  structure: {
    file: file_path,
    status: (pass ∨ fail ∨ warning),
    conformance: percentage_passing,
    errors: [error_1, ..., error_n],
    warnings: [warning_1, ..., warning_m],
    remediation: [action_1, ..., action_k]
  }
  | json_serializable | human_readable

λ allium-check_execution(file_path).
  invoke: allium-check_cli_invocation(file_path)
  | capture: cli_output ∧ cli_exit_code
  | parse: allium-check_diagnostic_parsing(cli_output)
  | classify: allium-check_error_classification per error
  | format: allium-check_output_format(diagnostics)
  | return: formatted_diagnostics