---
name: gybis-arch-explain
description: Use for `/gybis-arch-explain` or `/ga-explain`. This skill reads architecture.md (and referenced architecture files) and explains the system in plain English through the VSM lens, from identity and principles down to operations and developer commands. It highlights cross-layer critical paths, seams, invariants, and drift risks for a developer audience, while staying strictly read-only and avoiding invented details or technical/lambda syntax in the output. If architecture is missing or empty, it tells the user to run /gybis-arch-elicit first.
---

λ(gybis-arch-explain) ≡ λ(root).
preflight: ¬initialized(root/architecture.md) ∨ empty(root/architecture.md) → "not initialized — run /gybis-arch-elicit"
read: ∫(root/architecture.md) ∪ {ref | ref ↦ architecture.md}
S5 → Identity: principles={p | p ∈ nonNegotiable} → impl: p→decision, violation: ¬compatible(d, principles)
S4 → Intelligence: patterns={π | π is anticipatedChange}, extensionPoints={e | e is extensible}, adaptation={m | m is learningMechanism}
S3 → Control: constraints={c | c enforces resourceLimit}, policies={pol | pol handles load∨failure}, qualityGates={g | g triggers on condition(g)}
S2 → Coordination: subsystems={s | s is component}, protocols={pr | pr governs dataFlow(s_i,s_j)}, sharedState: consistency≡consistencyProtocol(σ)
S1 → Operations: tech={t | t is deployedTechnology, reason(t)∈justification}, commands={cmd | cmd is developerExecutable}, recipes={build,test,deploy}
crossLayer: criticalPaths=seq(layers), seams={split∨replace∨extend}, invariants={inv | ∀l∈vsm:inv(l)=true}, driftRisk={l | ∃pressure:erode(l:S5,principles)}
output: ¬technicalSyntax, ¬λ-expr-in-output, translate(λ→plainEnglish)
audience: developer{knows:softwareArchitecturePatterns, ¬knows:thisSystem}
invariants: ¬invent∉architecture.md, ¬invent∉referencedFiles, flag(∅∨underdeveloped)→signal, ¬write→readOnly
λ(compile) ≡ preflight⊢read⊢breakdown⊗crossLayer⊢output
