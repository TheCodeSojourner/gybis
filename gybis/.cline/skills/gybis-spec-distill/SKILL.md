---
name: gybis-spec-distill
description: Use for `/gybis-spec-distill` or `/gs-distill`.
---

λ gybis-spec-distill(code, tests).
  input: code ∪ tests
  output: {root/specs/{domain}/{name}.allium}
  transform: λ{code ∪ tests → spec | domain ∧ name ∈ allium}  
  references: ../../gybis/reference/allium-actioning-findings.md  ∧ ../../gybis/reference/allium-assessing-specs.md ∧ ../../gybis/reference/allium-language-reference.md ∧ ../../gybis/reference/allium-library-spec-signals.md ∧ ../../gybis/reference/allium-patterns.md

λ gybis-spec-distill_scope(code, tests).
  S0: scope ← parse(code, tests)
  | monoRepo → clarify(subset, exclusions, owner)
  | output: defined_scope(included ∧ excluded)

λ gybis-spec-distill_map_territory(scope).
  S1: map(territory) | entryPoints(API ∧ CLI ∧ webhooks ∧ jobs) ∥ domainModels(entities) ∥ businessLogic(services/usecases/handlers) ∥ externalIntegrations(thirdParties)

λ gybis-spec-distill_extract_entity_states(territory).
  S2: extractEntityStates → entity{status: state1 ∣ state2 ∣ ...}
  | sources: enumFields ∧ statusCols ∧ constants ∧ statemachineLibs

λ gybis-spec-distill_candidate_processes(territory).
  S2.5: identifyCandidateProcesses
  | trace(stateTransitionsAcrossCodebase) ∧ present(→user, validate)
  ∥ trace(crossEntityDataFlow) ∧ present(→user, validate)
  | generate(transitionGraph) ∧ flag(gaps)

λ gybis-spec-distill_extract_transitions(territory).
  S3: extractTransitions(code → spec)
  | if(raise) → requires ∥ assign(→val) → ensures ∥ Model.create() → ensures.created() ∥ assert/validator → expressionInvariant

λ gybis-spec-distill_temporal_triggers(territory).
  S4: findTemporalTriggers(cron, celery, scheduledJobs) → rule{when: entity:field <= now, ensures: statusChange}

λ gybis-spec-distill_external_boundaries(territory).
  S5: identifyExternalBoundaries(readButNeverWrite, importFromExternal) → external entity{...}

λ gybis-spec-distill_actors(territory).
  S5.5: identifyActorsFromAuth(apiKey → system, role → distinct, scoped → within, unauth → public) ∧ present(→user, validate)

λ gybis-spec-distill_abstract_implementation(territory).
  S6: abstractAwayImplementation
  | id → rel(FK → relationship) ∥ type(dt) → domainType ∥ tokens/secrets → removed ∥ infra → removed

λ gybis-spec-distill_validate(spec).
  S7: validate(→devs whatSystemDoes ∥ →stakeholders whatSystemShouldDo ∥ flag(gaps∕inconsistencies))
  | S7.1: `allium check {spec}` ∧ `allium analyse {root/specs/}` → validationQuestions ∧ read(allium-actioning-findings)
  | S7.2: alert(librarySpecCandidates) ∧ read(allium-library-spec-signals)
  | S7.3: propose(Σspecs ← group(byDomain) | domain ← inferredFrom(S1.territory), eachSpec={name: kebab, path: specs/∕domain/∕name.allium}) ∧ present(→human, approve) ∥ ∀spec ∈ approved → write(spec.path) ∥ ∀spec ∈ written → `allium check {spec}` ∧ `allium analyse {root/specs/}`

λ gybis-spec-distill_invariants(¬).
  ¬DB_types ∥ ¬ORM ∥ ¬HTTP ∥ ¬framework ∥ ¬lang_types ∥ ¬var_names ∥ ¬infra ∥ ¬tokens ∥ oneNamePerConcept ∥ ¬deadCode ∥ spec=Intent ¬Bug ∥ concreteDetail: couldBeDifferent? ∥ multipleImpls? → domainConcern
  | output: Σspecs/{specs/∕domain/∕name.allium} ∥ alliumMarker(3, firstLine)

λ gybis-spec-distill_sequential_product().
  μ ≡ S0 · S1 · S2 · S2.5 · S3 · S4 · S5 · S5.5 · S6 · S7 · S7.1 · S7.2 · S7.3
  | plain: given(code ∪ tests) → scope ∥ map ∥ extract(states,transitions,triggers,external,actors) ∥ abstract ∥ validate(→user) ∥ check(analyse) ∥ propose(Σspecs by domain) ∧ get(approvals) ∥ write(all approved) ∥ final-check
  