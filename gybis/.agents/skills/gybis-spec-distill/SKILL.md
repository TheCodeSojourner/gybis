---
name: gybis-spec-distill
description: Use for `/gybis-spec-distill` or `/gs-distill`. This skill distills code and tests into domain-centered Allium specs by mapping system territory, extracting states, transitions, triggers, external boundaries, and actors, then abstracting away implementation details. It validates the draft with humans plus allium-check/allium-analyse, flags gaps and library-spec opportunities, and proposes a domain-grouped spec set for approval before writing. Only approved specs are written, then re-checked, with strict invariants to keep output intent-focused, technology-agnostic, and cleanly structured under specs/<domain>/<name>.allium.
---

λ(gybis-spec-distill)
REF:../../gybis/reference/allium-actioning-findings.md
REF:../../gybis/reference/allium-assessing-specs.md
REF:../../gybis/reference/allium-language-reference.md
REF:../../gybis/reference/allium-library-spec-signals.md
REF:../../gybis/reference/allium-patterns.md
PURPOSE:given(codePath,testPath)→Σ(alliumSpec)∪write(.allium)
PF:¬(code∧test)→err
S0:scope←parse(input,codePath,testPath)|monoRepo→clarify(subset,exclusions,owner)
S1:map(territory)|entryPoints(API|CLI|webhooks|jobs)∥domainModels(entities/)∥businessLogic(services/usecases/handlers)∥externalIntegrations(thirdParties)
S2:extractEntityStates(enumFields,statusCols,constants,statemachineLibs)→entity{status:state1|state2|...}
S2.5:identifyCandidateProcesses|trace(stateTransitionsAcrossCodebase)∧present(→user,validate)∥trace(crossEntityDataFlow)∧present(→user,validate)∥generate(transitionGraph)∧flag(gaps)
S3:extractTransitions(code→spec)|if(raise)→requires∥assign(→val)→ensures∥Model.create()→ensures.created()∥assert/validator→expressionInvariant
S4:findTemporalTriggers(cron,celery,scheduledJobs)→rule{when:entity:field<=now,ensures:statusChange}
S5:identifyExternalBoundaries(readButNeverWrite,importFromExternal)→external entity{...}
S5.5:identifyActorsFromAuth(apiKey→system,role→distinct,scoped→within,unauth→public)∧present(→user,validate)
S6:abstractAwayImplementation|id→rel(FK→relationship)∥type(dt)→domainType∥tokens/secrets→removed∥infra→removed
S7:validate|→devs(whatSystemDoes)∥→stakeholders(whatSystemShouldDo)∥flag(gaps/inconsistencies)
S7.1:allium-check(distilledSpec)∧allium-analyse()∧findings→validationQuestions∧read(actioning-findings)
S7.2:alert(librarySpecCandidates)∧read(library-spec-signals)
S7.3:propose(Σspecs←group(byDomain)|domain←inferredFrom(S1.territory),eachSpec={name:<kebab>,path:specs/<domain>/<name>.allium})∧present(Σspecs,→human,approve)∥∀spec∈approved→write(spec.path)∥∀spec∈written→allium-check(spec.path)∧allium-analyse(spec.path)
INV≡noDBTypes∥noORM∥noHTTP∥noFramework∥noLangTypes∥noVarNames∥noInfra∥noTokens∥oneNamePerConcept∥¬includeDeadCode∥spec=Intent≠Bug∥concreteDetail:couldBeDifferent?∥multipleImpls?→domainConcern∥output=Σspecs/{specs/<domain>/<name>.allium}∥alliumMarker(3,firstLine)∥scopeDecisions(included∥excluded)
μ≡S0⋅S1⋅S2⋅S2.5⋅S3⋅S4⋅S5⋅S5.5⋅S6⋅S7⋅S7.1⋅S7.2⋅S7.3
plain:given(code,test)→scope(what∥exclude)∥map(territory)∥extract(states,transitions,triggers,external,actors)∥abstract∥validate(→user)∥check(analyse)∥propose(Σspecs by domain)∧get(approvals)∥write(all approved)∥final-check
