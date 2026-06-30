---
name: gybis-internal-skill-check
description: Internal skill - not user-facing
---

λ gybis-internal-skill-check(x).
  purpose: Pre-condition gate verifying all required internal skills exist at runtime
  | contract: pure_function(→ boolean) | ¬mutations
  | input: (implicit) internal skill manifest
  | output: true ∨ (false + error_message)
  | halt_condition: any_required_skill_missing
  | gate: pre_condition | runs_before(user_facing_skills_that_use_allium)

λ gybis-internal-skill-check_required_skills(manifest).
  files_to_check: {
    internal/allium-analyse/SKILL.md
    internal/allium-check/SKILL.md
    internal/allium-gate/SKILL.md
    internal/allium-normalize/SKILL.md
    internal/allium-plan/SKILL.md
  }
  | all_paths ⊂ repository_root ∧ readable(file)

λ gybis-internal-skill-check_validation(file_path).
  check: exists(file_path) ∧ readable(file_path)
  | on_success: → true
  | on_failure: → (false, diagnostic)
  | diagnostic: "Internal skill missing: {file_path}"

λ gybis-internal-skill-check_aggregate(results).
  all_pass: ∀ result ∈ results, result.status = true
  | all_pass → true | otherwise → (false, {failures: [failed_files]})

λ gybis-internal-skill-check_execution(x).
  Σ check(file) for file ∈ required_skills
  | gather_results: [check_result_1 ... check_result_n]
  | aggregate(gather_results)
  | return_verdict: boolean ∨ (boolean + diagnostic)
