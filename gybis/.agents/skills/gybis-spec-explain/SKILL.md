---
name: gybis-spec-explain
description: Use for `/gybis-spec-explain` or `/gs-explain`. This skill reads one or more Allium specs (file, domain, or all) and produces a technical, developer-focused explanation of domain structures, behaviors, constraints, guarantees, and cross-spec relationships. It outputs precise prose with light headings and domain terminology, while avoiding implementation code, pseudocode, and speculative design. It remains strictly read-only, surfaces gaps as technical questions, and reports clearly when specs are missing or empty.
---

λ(gybis-spec-explain)
REF:../../gybis/reference/allium-language-reference.md
PURPOSE:given(file|domain)→prose(technical,developer)∪readOnly
PF:¬alliumFiles→say(emptyOrMissing)
S0:files←parse(input)|file→read(single)∥domain→read(all,<root>/specs/{domain}/*.allium)∥noArg→read(all,<root>/specs/**/*.allium)∥ambiguous→ask(human,clarify)
S1:narrative←Σ{
  structures:domainConcepts(responsibilities,boundaries)∧specTerminology(precision)∥conceptParticipation(modeledBehavior),
  behaviors:actions∪events∪stateTransitions∥conditions∥observableOutcomes∪sideEffects,
  constraints:preconditions∪validationRules∪stateDependentConstraints∥permits∪requires∪rejects∥boundaryConditions∪exceptions∪invalidTransitions,
  guarantees:invariants(alwaysTrue)∥ordering∪consistencyRules∪irreversibleTransitions∥terminalLockedStates,
  domainMode:adjacentSpecs(relations)∥sharedConcepts∪handoffs∪dependencies∪lifecycleContinuity
}
S2:output(technicalProse,lightHeadings,preciseDomainTerminology,¬implCode,¬pseudocode,¬speculativeDesign)
INV≡∀behaviors∪stateTransitions∪constraints∪guarantees→output∥¬omit∥multiFile→coherentTechnicalNarrative∥gaps→technicalQuestions(human)∥readOnly∥empty→saySo∥audience=developer(techLead,architect)
μ≡S0⋅S1⋅S2
plain:given(scope)→resolve(files)∥read(inFull)∥translate(technicalNarrative)∥output(prose,lightHeadings,specTerminology)∥surface(gaps)
