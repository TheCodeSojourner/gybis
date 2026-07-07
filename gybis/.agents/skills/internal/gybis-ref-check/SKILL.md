---
name: gybis-ref-check
description: Internal skill - not user-facing
---

λ gybis-ref-check(x).
  purpose: Pre-condition gate verifying all required upstream reference files exist at runtime
  | contract: pure_function(→ boolean) | ¬mutations
  | input: (implicit) reference file manifest
  | output: true ∨ (false + error_message)
  | halt_condition: any_required_file_missing
  | gate: pre_condition | runs_before(all_user_facing_skills)

λ gybis-ref-check_reference_roots(roots).
  roots_to_check: {
    internal/reference
    .agents/skills/internal/reference
  }
  | rule: prefer(internal/reference) when both_exist
  | fallback: use(any_existing_root)
  | diagnostic_on_none: "No valid reference root found. Checked: internal/reference, .agents/skills/internal/reference"

λ gybis-ref-check_required_files(manifest).
  files_to_check: {
    allium-assessing-specs.md
    allium-language-reference.md
    allium-patterns.md
    allium-recommended-loops.md
    allium-constructs.md
    allium-actioning-findings.md
    allium-library-spec-signals.md
    vsm-guide.md
  }
  | all_paths resolved_via(reference_roots) ∧ readable(file)

λ gybis-ref-check_validation(file_name, roots).
  check: ∃ root ∈ roots, exists(root/file_name) ∧ readable(root/file_name)
  | on_success: → true
  | on_failure: → (false, diagnostic)
  | diagnostic: "Reference file missing: {file_name} (checked roots: internal/reference, .agents/skills/internal/reference)"

λ gybis-ref-check_aggregate(results).
  all_pass: ∀ result ∈ results, result.status = true
  | all_pass → true | otherwise → (false, {failures: [failed_files]})

λ gybis-ref-check_execution(x).
  resolve_roots(reference_roots)
  | require(roots_found) ∨ return(false, diagnostic_on_none)
  | Σ check(file_name, roots) for file_name ∈ required_files
  | gather_results: [check_result_1 ... check_result_n]
  | aggregate(gather_results)
  | return_verdict: boolean ∨ (boolean + diagnostic)