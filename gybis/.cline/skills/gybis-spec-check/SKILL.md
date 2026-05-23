---
name: gybis-spec-check
description: Use for `/gybis-spec-check` or `/gs-check`.
---

λ gybis-spec-check(input).
  λ workflow(ξ).
    input: ξ ∈ {file | domain | scope | ∀}
    output: clean ∨ issues(Σ) → fixes(Φ) → clean
    composition: S0 → S1 → S2 → S3
  λ prerequisite(ξ).
    ¬∃allium(ξ) → error ∧ terminate
    | allium_found → proceed
  λ S0_parse(ξ).
    resolve(ξ) → files(Σ)
    | ξ = (domain, name) → specs/{domain}/{name}.allium
    | ξ = domain → specs/{domain}/*.allium
    | ξ = ⊥ → specs/*/*.allium
    | Σ = {f₁, f₂, ..., fₙ}
  λ S1_check(Σ).
    ∀f ∈ Σ → S2(f)
  λ S2(f).
    S2.1_run(f) → {clean | issues | not_found}
  λ S2.1_run(f).
    allium_check(f) → exit_code
    | exit 0 → clean(f) → S1(next)
    | exit 1 → diagnostics(D) → group(severity) → apply(actioning) → issues(Σᵢ) → S2.2
    | exit 2 → report(not_found, operator) → S1(next)
  λ S2.2(Σᵢ).
    sort(Σᵢ, domain → name → priority) → numbered_list
    | present(Σᵢ, operator) → approvals(Φ)
    | apply(Φ, codebase) → S2.1_run(f)
  λ invariants.
    I₁: cli_primary | workflow ≡ CLI_interface
    I₂: human_in_loop | ∀d ∈ D → decision(human, d) ∧ ¬auto_action
    I₃: errors_block_writes | errors(E) ∧ ¬resolved(E) → ¬modifications
    I₄: warnings_need_decision | warnings(W) → review(human) → action
    I₅: revalidation | apply_fixes → recheck → ¬regressions
  λ composition.
    S0(ξ) → S1(Σ) → S2(Σ) → S3(clean)
  λ summary(ξ).
    read(ξ) → resolve(allium_files)
    | run(allium_check) → parse(diagnostics)
    | group(severity) → present(numbered_issues)
    | get(approvals) → apply(fixes)
    | recheck → exit 0 → report(clean)
    