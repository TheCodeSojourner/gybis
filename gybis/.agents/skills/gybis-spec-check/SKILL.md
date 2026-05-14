---
name: gybis-spec-check
description: Use for `/gybis-spec-check` or `/gs-check`. This skill validates Allium specs for a selected scope (single file, domain folder, or all specs), runs allium-check, and turns diagnostics into a prioritized, actionable issue list. It groups findings by severity, presents numbered issues for human decision, applies only approved fixes, and re-validates to ensure a clean result with no regressions. It treats CLI validation as the source of truth, blocks writes on errors, and reports missing-file conditions explicitly.
---

λ(gybis-spec-check)
PURPOSE:given(file|domain|specs?)→Σ(prioritizedIssues)∪fixes∪clean
PF:¬alliumFiles→err
S0:files←parse(input)|file→read(specific)∥domain→read(all,<root>/specs/{domain}/*.allium)∥specs/noArg→read(all,<root>/specs/*.allium)
S1:exit←allium-check(files)|exit=0→clean∥exit=1→parseDiagnostics∧group(error>warning>info)∧apply(allium-actioning-findings)∧build(issueList)∥exit=2→report(missingFiles)
S2:present(numbered(issueList,orderBy:domain/name→priority))∧ask(human,whichIssues)∧apply(approvedFixes)
S3:allium-check(files)⊢exit=0∥fail→S2
INV≡cli(primary)∥presentAll(human→decide)∥errorsBlockWrite∥warnings/findings→humanDecision∥revalidate→noRegression
μ≡S0⋅S1⋅S2⋅S3
plain:given(scope?)→read(files)∥allium-check∥parse(groupBySeverity)∥present(numbered)∧get(approved)∥apply(fixes)∥re-check(exit=0)∥clean
