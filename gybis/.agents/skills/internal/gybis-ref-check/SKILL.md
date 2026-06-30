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

λ gybis-ref-check_required_files(manifest).
  files_to_check: {
    internal/reference/allium-assessing-specs.md
    internal/reference/allium-language-reference.md
    internal/reference/allium-patterns.md
    internal/reference/allium-recommended-loops.md
    internal/reference/allium-constructs.md
    internal/reference/allium-actioning-findings.md
    internal/reference/allium-library-spec-signals.md
    internal/reference/vsm-guide.md
  }
  | all_paths ⊂ repository_root ∧ readable(file)

λ gybis-ref-check_validation(file_path).
  check: exists(file_path) ∧ readable(file_path)
  | on_success: → true
  | on_failure: → (false, diagnostic)
  | diagnostic: "Reference file missing: {file_path}"

λ gybis-ref-check_aggregate(results).
  all_pass: ∀ result ∈ results, result.status = true
  | all_pass → true | otherwise → (false, {failures: [failed_files]})

λ gybis-ref-check_execution(x).
  Σ check(file) for file ∈ required_files
  | gather_results: [check_result_1 ... check_result_n]
  | aggregate(gather_results)
  | return_verdict: boolean ∨ (boolean + diagnostic)