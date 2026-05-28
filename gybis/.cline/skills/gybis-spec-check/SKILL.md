---
name: gybis-spec-check
description: Use for `/gybis-spec-check` or `/gs-check`.
---

λ gybis-spec-check(input).
  pipeline(S0 → S1 → S2 → S3) | clean_output ∧ issue_detection ∧ fix_application
  | input ∈ {domain_concern, domain, all_specs} | default: all_specs

λ gybis-spec-check_resolved_paths(input).
  domain_concern → root/specs/{domain}/{concern}.md
  | domain → root/specs/{domain}/*.md
  | all_specs → root/specs/*/*.md

λ gybis-spec-check_prerequisites(x).
  gate(cli_available ∧ cli_version_satisfies ∧ input_resolves) | ¬all_gates → halt
  | cli_available: allium --version | ¬available → recommend(https://github.com/juxt/allium_tools) ∧ halt
  | cli_version_satisfies: version(allium) ≥ 3 | ¬satisfies → recommend(https://github.com/juxt/allium_tools) ∧ halt
  | input_resolves: ∃files(root/specs/**/*.allium) | ¬∃ → notify(human) ∧ halt

λ gybis-spec-check_pipeline(input).
  S0(input) → S1() → S2(file_set) → S3()

λ gybis-spec-check_S0(input).
  input ∈ {⊥, "all", "all specs"} → root/specs/**/*.allium
  | input ∈ {domain, concern} → root/specs/{domain}/{concern}.allium
  | input ∈ {domain} → root/specs/{domain}/*.allium
  | result: {f₁, f₂, ..., fₙ}

λ gybis-spec-check_S1(file_set).
  ∀f ∈ file_set → S2(f)
  | ¬∃file_set → notify(human) ∧ halt

λ gybis-spec-check_S2(f).
  outcome(allium_check(f)) ∈ {clean, issues, not_found}
  | clean → next(file)
  | issues → S2.2(diagnostics)
  | not_found → report(not_found) ∧ next(file)

λ gybis-spec-check_S2.1(f).
  exit_code(allium_check(f)) ∈ {0, 1, 2}
  | 0 → next(file)
  | 1 → collect(diagnostics) ∧ group(by_severity) ∧ action_items → S2.2
  | 2 → report(not_found) ∧ next(file)

λ gybis-spec-check_S2.2(diagnostics).
  sort(diagnostics, by(domain, name, priority)) → numbered_list
  | present(numbered_list, human) → approval
  | apply(approved_fixes, codebase)
  | recheck(f) → S2.1(f)
  | loop: ¬clean(f)

λ gybis-spec-check_S3(file_set).
  ∀f ∈ file_set → clean ∧ ¬regressions
  | report(success, exit_0)

λ gybis-spec-check_invariant_I₁.
  cli_primary_interface | ∀workflow use(cli)

λ gybis-spec-check_invariant_I₂.
  ∀diagnostic d → requires(human_approval) | ¬automatic

λ gybis-spec-check_invariant_I₃.
  errors_exist(E) ∧ ¬resolved(E) → ¬permission(modify)

λ gybis-spec-check_invariant_I₄.
  warnings_exist(W) → requires(human_review) ∧ requires(human_action)

λ gybis-spec-check_invariant_I₅.
  apply(fixes) → recheck(file) → confirm(¬regressions)

λ gybis-spec-check_composition.
  S0(parse) → S1(iterate) → S2(check ∧ fix) → S3(report_clean)

λ gybis-spec-check_summary.
  parse(input) → resolve(allium_files)
  | run(allium_check, fᵢ) → parse(diagnostics)
  | group(diagnostics, severity) → present(numbered_list, human)
  | retrieve(approvals) → apply(fixes, codebase)
  | recheck(modified) → confirm(¬regressions)
  | ∀clean → exit(0) ∧ report(success)
  
    