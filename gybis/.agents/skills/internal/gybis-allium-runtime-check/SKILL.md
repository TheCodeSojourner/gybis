---
name: gybis-allium-runtime-check
description: Internal skill - not user-facing
---

λ gybis-allium-runtime-check(target = none).
  purpose: Verify the installed Allium executable before adapter dispatch
  | contract: pure_function(target → true ∨ false + diagnostic) | ¬mutations
  | input: target ∈ {none ∨ spec_file ∨ specs_path}
  | output: true ∨ (false + diagnostic)
  | gate: pre_condition | runs_before(gybis-allium-check ∧ gybis-allium-analyse ∧ gybis-allium-plan)

λ gybis-allium-runtime-check_cli_surface(x).
  allowed_cli: {allium check, allium analyse, allium plan, allium parse, allium model}
  | prohibited_cli: {allium gate, allium rerun} | rationale: gybis-allium-* are gybis skills, ¬allium subcommands

λ gybis-allium-runtime-check_version().
  invoke: "allium --version"
  | parse: executable_version ≔ semver(output)
  | supported: executable_version ≥ 3.5.3
  | missing ∨ unparseable ∨ unsupported
    → false + "Unsupported Allium executable: expected version >= 3.5.3"

λ gybis-allium-runtime-check_target(target).
  target = none → true
  | target = spec_file → ∃(target) ∧ file_type(target) = .allium
  | target = specs_path → ∃(target) ∧ is_directory(target)
  | invalid_target → false + "Allium target is missing or invalid: {target}"

λ gybis-allium-runtime-check_spec_inventory(specs_path).
  spec_files ≔ recursive_files(specs_path, extension = .allium)
  | card(spec_files) = 0
    → false + "NO_SPECS: no .allium files found under {specs_path}"
  | otherwise → true

λ gybis-allium-runtime-check_payload(target).
  target = none → true
  | target = spec_file
    → validate_json(allium check target, required = {diagnostics, findings, spec_file})
    ∧ validate_json(allium plan target, required = {diagnostics, obligations, version})
  | target = specs_path
    → invoke(gybis-allium-runtime-check_spec_inventory(target))
    ∧ validate_json(allium check target, required = {diagnostics, findings})
    ∧ validate_json(allium analyse target, required = {diagnostics, findings})
  | malformed_json ∨ missing_required_field ∨ incompatible_field_shape
    → false + "Allium JSON adapter contract is incompatible for {target}"
  | exit_code ∈ {0, 1} ∧ valid_json → true
  | exit_code > 1 ∨ invocation_failure → false + structured_failure

λ gybis-allium-runtime-check_execution(target = none).
  invoke(gybis-allium-runtime-check_cli_surface()) → true
  | invoke(gybis-allium-runtime-check_version()) → true ∨ return(false, diagnostic)
  | invoke(gybis-allium-runtime-check_target(target)) → true ∨ return(false, diagnostic)
  | invoke(gybis-allium-runtime-check_payload(target)) → true ∨ return(false, diagnostic)
  | return(true)

λ gybis-allium-runtime-check_regression_contract(x).
  invariant: unsupported_or_missing_executable → false + diagnostic
  | invariant: target = none ∧ compatible_executable → true
  | invariant: specs_path ∧ card(.allium_files) = 0 → false + NO_SPECS
  | invariant: valid_json ∧ exit_code ∈ {0, 1} → not_malformed
  | invariant: missing_required_field → false + diagnostic
  | invariant: zero_file_mutations