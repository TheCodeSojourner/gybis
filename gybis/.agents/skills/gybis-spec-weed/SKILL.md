---
name: gybis-spec-weed
description: This skill finds and resolves divergence between an Allium spec and implementation by combining CLI analysis, model-based spec parsing, and code trace inspection. It classifies mismatches (aligned, partial, missing, contradicted), includes process-level issues like deadlocks or unreachable flows, and presents two fix directions for each divergence: align code to spec or spec to code. It never resolves silently, requires human choice per item, and re-validates with allium-check/allium-analyse after approved changes.
---

λ(gybis-spec-weed)
REF:../../reference/allium-actioning-findings.md
PURPOSE:given(specFile)→Σ(divergences)∪humanDecides(fixDirection)∪apply(fix)
PF:¬alliumSpec→err
S0:allium-check(specFile)∧allium-analyse(specFile)∥read(findings JSON)∥process-level(deadlocks,conflicts,unreachable,dataFlowGaps)→weedReport(specInternalDivergences)
S1:allium-model(specFile)∥extract(entities[].fields,entities[].transitions)∥use(modelJSON,notSpecProse)for systematicComparison
S2:readImplementation|follow(traces:references)∥search(codebase,ruleNames+entities)for rules without traces
S3:divergenceAnalysis|∀rule∈spec:aligned(implementation matches ensures)∥partial(some ensures covered)∥missing(no impl found)∥contradicted(impl does spec prohibits)∥∀field∈modelJSON:spec→code(absent)∥code→spec(absent)∥transition(code∉transitionGraph)∥∀invariant:codePath(violation)
S4:weedReport|CLI findings(deadlock→ask human:terminal?)∥specCodeDivergences(rule:contradicted/partial/missing)∥entityDivergences(field present/absent per direction)∥∀item→proposedFix A(matches spec)∥∀item→proposedFix B(matches code)
S5:humanDecides|∀divergence→choose(fix A∥fix B∥other)∥AI applies(approved fix to spec∥code)∥∀spec change→allium-check∧allium-analyse→confirm(health)
INV≡allium analyse before code read∥allium model for field comparison∥¬silentResolve(∀fix→human decision)∥code≠spec(both equally valid starting points)∥postFix→full validation∥CLI findings⊂weed report
μ≡S0⋅S1⋅S2⋅S3⋅S4⋅S5
plain:given(spec)→check+analyse∥model∥read(impl)∥analyze(divergences)∥report(findings+divergences+A+B)∥human(decide)∥apply(fix)∥revalidate
