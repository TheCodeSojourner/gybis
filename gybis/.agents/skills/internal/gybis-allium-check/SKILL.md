---
name: gybis-allium-check
description: Internal skill - not user-facing
---

λ gybis-allium-check(file_path).
  purpose: Provide pure per-file diagnostics for .allium conformance
  | contract: pure_function(file_path → diagnostics) | ¬mutations
  | input: path to .allium file
  | output: diagnostics + status + errors ∨ warnings
  | constraint: read_only | caller_mutates_files
  | precondition: file_path ∧ ∃(file_path) ∧ file_type = .allium

λ gybis-allium-check_cli_invocation(file_path).
  cli_command: "allium check {file_path}"
  | execution: deterministic(file_path) → fixed_output
  | stderr_handling: capture_and_return
  | stdout_handling: parse_json_envelope

λ gybis-allium-check_diagnostic_parsing(cli_output).
  parse: cli_output → {command, spec_file, diagnostics[], findings[]}
  | diagnostics: line_level_structural_warnings_and_errors
  | findings: present_in_envelope ∧ may_be_empty
  | diagnostic.severity ∈ {error, warning, info, Error, Warning, Info}
  | diagnostic.code: string ∨ null
  | diagnostic.location: {file, line, col} ∨ null
  | exit_code ∈ {0, 1} does_not_override_valid_json
  | malformed_json ∨ exit_code > 1 → structured_failure

λ gybis-allium-check_error_classification(error).
  type: {syntax_error, semantic_error, validation_error, unknown_error} | severity: {critical, high, medium, low} | line_number: error.location.line | message: error.message

λ gybis-allium-check_output_format(diagnostics).
  structure: {file: spec_file, status ∈ (pass ∨ fail ∨ warning), conformance, errors: [...], warnings: [...], remediation: [...], raw_diagnostics: [...]}

λ gybis-allium-check_execution(file_path).
  invoke(gybis-allium-runtime-check_version()) → true ∨ halt("Allium runtime compatibility check failed")
  | invoke: gybis-allium-check_cli_invocation(file_path)
  | capture: cli_output ∧ cli_exit_code
  | parse: gybis-allium-check_diagnostic_parsing(cli_output)
  | classify: gybis-allium-check_error_classification per diagnostic
  | format: gybis-allium-check_output_format(diagnostics)
  | return: formatted_diagnostics