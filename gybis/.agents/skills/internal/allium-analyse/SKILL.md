---
name: allium-analyse
description: Internal skill - not user-facing
---

λ allium-analyse_shell_guard(x).
  classification: internal_skill_alias(allium-gate) ∧ ¬shell_subcommand(allium gate)
  | shell_prohibition: ¬execute("allium gate") ∧ ¬execute("allium rerun")
  | allowed_cli: {allium check, allium analyse, allium plan, allium parse, allium model}

λ allium-analyse(specs_path).
  purpose: Provide pure set-level diagnostics for the specs/ directory
  | contract: pure_function(specs_path → findings) | ¬mutations
  | input: path to specs/ directory
  | output: findings + status + consistency_checks
  | constraint: read_only | no_mutations
  | precondition: specs_path ∧ exists(specs_path) ∧ is_directory(specs_path)

λ allium-analyse_cli_invocation(specs_path).
  cli_command: "allium analyse {specs_path}"
  | execution: deterministic(specs_path) → fixed_output
  | stderr_handling: capture_and_return
  | stdout_handling: parse_json_envelope

λ allium-analyse_scope(specs_path).
  scope: ∀ .allium_files ∈ specs_path ∧ subdirectories
  | recursion: depth = unbounded | all_depths_analyzed
  | relationships: inter_spec_dependencies ∧ pattern_consistency

λ allium-analyse_finding_types(findings).
  categories: {dependency_issues, pattern_violations, semantic_issues, structural_issues} ∨ finding.type
  | severity: {critical, high, medium, low, informational}

λ allium-analyse_output_format(findings_data).
  structure: {directory: specs_path, status ∈ (pass ∨ findings_detected ∨ critical_issues), total_specs, diagnostics: [...], findings: [...], summary: {critical, high, medium, low, informational}, remediation: [...]}
  | json_serializable | human_readable

λ allium-analyse_execution(specs_path).
  invoke(allium-runtime-check_version()) → true ∨ halt("Allium runtime compatibility check failed")
  invoke(allium-analyse_shell_guard) → true
  | invoke: allium-analyse_cli_invocation(specs_path)
  | capture: cli_output ∧ cli_exit_code
  | parse: allium-analyse_findings_parsing(cli_output) → {diagnostics, findings, spec_file}
  | classify: allium-analyse_finding_classification per finding
  | format: allium-analyse_output_format(findings_data)
  | return: formatted_findings