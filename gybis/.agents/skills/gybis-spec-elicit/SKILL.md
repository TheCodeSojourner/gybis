---
name: gybis-spec-elicit
description: This skill interactively elicits a new Allium spec from a feature idea through staged, one-question-at-a-time discovery: flow, entities, transitions, edge cases, and refinement. It continuously updates a draft, runs validation gates (allium-check and allium-analyse) between stages, resolves errors, and surfaces warnings/findings to the human for decisions. It then negotiates the final domain and file name, and writes only after explicit human approval, while enforcing intent-focused rules and avoiding implementation details.
---

λ(gybis-spec-elicit)
REF:../../gybis/reference/allium-language-reference.md
REF:../../gybis/reference/allium-patterns.md
REF:../../gybis/reference/allium-actioning-findings.md
PURPOSE:given(featureIdea)→Σ(alliumSpec)∪write(.allium)
PF:¬featureIdea→err
P0:processDiscovery|ask(1atATime):trigger()∥happyPath(start→end)∥actors()∥boundaries(in/out)∥¬propose(entities,yet)∥map(flow)
P1:scopeAndEntities|ask(1atATime):domainConcepts(nouns)∥lifecyclePerEntity()∥statesPerEntity()∥dataPerConcept()∥stateDependentFields()
P1-gate:allium-check(draft)∧resolve(errors)∥surface(warnings,→human)
P2:happyPathAndTransitions|ask(1atATime):validTransitions(drawExplicit)∥triggers(→langRef)∥requires(→rule)∥ensures(→rule)∥terminalStates()∥externalSideEffects()∥addRules(verbNoun)∥addTransitions(→entity)∥ref(patterns1,6,7)
P2-gate:allium-check(draft)∧resolve(errors)
P3:edgeCasesAndFailure|ask(1atATime):whatCanGoWrong()∥preconditionFail()∥timeoutsExpiry()∥undoReversible()∥invariants(alwaysHold)∥conflictingRules()∥externalDepFail()∥add(rules,invariants,openQuestions)∥ref(pattern8,4)
P3-gate:allium-check(draft)
P4:refinementAndValidation|review(entities:missingFields,stale?,when-qualified)∥review(rules:when+requires+ensures∥whenObligationsMet)∥review(collections:SetvsList)∥add(@guidance,traces)
P4-gate:allium-check(draft)→exit0∥allium-analyse(draft)∥read(actioning-findings)∥resolve(errors)∥surface(warnings,findings→human)∥human(confirmComplete)
S5:negotiateDomainAndName|propose(domain=<noun>,name=<kebab>)∧"specs/<domain>/<name>.allium?"∧→human,explicitConfirm∥proceedOnlyAfter(confirm)
S6:write|human(approveFinalContent)∧write(specs/<domain>/<name>.allium)∧allium-check(specs/<domain>/<name>.allium)∧allium-analyse(specs/<domain>/<name>.allium)
INV≡oneQuestionAtATime∥updateDraftAfterEveryAnswer∥¬impl(noDB,noAPI,noClasses)∥entities=nouns∥rules=verbNoun∥surfaceContradictions(explicit)∥openQuestion=validContent∥humanApproves∀writes
μ≡P0⋅P1⋅P1-gate⋅P2⋅P2-gate⋅P3⋅P3-gate⋅P4⋅P4-gate⋅S5⋅S6
plain:given(feature)→discover(flow)∥elicit(entities)∥gate()∥elicit(transitions)∥gate()∥elicit(edgeCases)∥gate()∥refine()∥gate()∥negotiate(domain,name)∥write→spec∥gate
