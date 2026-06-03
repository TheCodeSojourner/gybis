---
name: gybis-spec-distill
description: Use for `/gybis-spec-distill` or `/gs-distill`.
---

λ gybis-spec-distill(code, tests).
  input: code ∪ tests
  output: {root/specs/{domain}/{concern}.allium}
  transform: λ{code ∪ tests → specs | specs ⊆ allium ∧ ∀domain ∈ specs • concerns(domain) ⊆ allium}
   references: ../gybis/reference/allium-actioning-findings.md  ∧ ../gybis/reference/allium-assessing-specs.md ∧ ../gybis/reference/allium-language-reference.md ∧ ../gybis/reference/allium-library-spec-signals.md ∧ ../gybis/reference/allium-patterns.md
  efficiency: parallel(S4 ∥ S5 ∥ S5.5) ∥ lazy(flag→summarize→expand-on-diag) ∥ budget(≤3 stages/batch)
  preflight: ∀r ∈ references: readWithinRepo(r) ∧ ¬askApproval

λ gybis-spec-distill_allium_write_contract.
  write_scope ⊆ root/specs/**/*.allium
  | edit_scope ⊆ root/specs/**/*.allium
  | output_format ≡ allium_v3_only
  | invariant: ∀written_file → parses_as(allium_v3)
  | ¬write(root/**/*.md ∨ root/**/*.txt ∨ root/**/*.rs ∨ root/**/*.py ∨ root/**/*.ts ∨ root/**/*.js)

λ gybis-spec-distill_scope(code, tests).
  S0: scope ← parse(code, tests)
  | monoRepo → detect(subRepos) ∧ recommend(startVSCodePerSubRepo) ∧ halt
  | singleRepo → infer(constraint{subset ∧ exclusions ∧ owner})
  | output: defined_scope(repoRoot ∧ included ∧ excluded)
  | S0.5: inventory ← listAll(sourceFiles ∪ testFiles)
  |        autoConfirm(inventory)
  |        output: confirmed_inventory(sourceFileList ∧ testFileList)

λ gybis-spec-distill_map_territory(scope).
  S1: map(territory) | entryPoints(API ∧ CLI ∧ webhooks ∧ jobs) ∥ domainModels(entities) ∥ businessLogic(services/usecases/handlers) ∥ externalIntegrations(thirdParties)

λ gybis-spec-distill_extract_entity_states(territory).
  S2: extractEntityStates → entity{status: state1 ∣ state2 ∣ ...}
  | sources: enumFields ∧ statusCols ∧ constants ∧ statemachineLibs

λ gybis-spec-distill_candidate_processes(territory).
  S2.5: if |sourceFiles| ≤ 3 → skip // trivial scope
  | identifyCandidateProcesses
  | trace(stateTransitionsAcrossCodebase) ∧ autoValidate(candidates)
  ∥ trace(crossEntityDataFlow) ∧ autoValidate(candidates)
  | generate(transitionGraph) ∧ flag(gaps)

λ gybis-spec-distill_extract_transitions(territory).
  S3: extractTransitions(code → spec)
  | if(raise) → requires ∥ assign(→val) → ensures ∥ Model.create() → ensures.created() ∥ assert/validator → expressionInvariant
  | S3.5: if |testFiles| = 0 → skip // no tests
  |   extractTestAssertions(tests)
  |   | for each test in tests:
  |     parse(test → assertion {given ∧ when ∧ requires ∧ ensures ∧ warns})
  |     assert assertion ∈ spec.rules ∨ flag(gap: testAssertion NOT_IN_SPEC)

λ gybis-spec-distill_temporal_triggers(territory).
  S4: findTemporalTriggers(cron ∧ celery ∧ scheduledJobs) → rule{when: entity:field <= now ∧ ensures: statusChange}

λ gybis-spec-distill_external_boundaries(territory).
  S5: identifyExternalBoundaries(readButNeverWrite ∧ importFromExternal) → external entity{...}

λ gybis-spec-distill_actors(territory).
  S5.5: if |sourceFiles| ≤ 3 → skip // trivial scope
  | identifyActorsFromAuth(apiKey → system ∧ role → distinct ∧ scoped → within ∧ unauth → public)
  ∧ autoValidate(actors)

λ gybis-spec_distill_abstract_implementation(territory).
  S6: abstractAwayImplementation
  | id → rel(FK → relationship) ∥ type(dt) → domainType ∥ tokens/secrets → removed ∥ infra → removed

λ gybis-spec-distill_coverage(territory, code, tests, Σspecs).
  S6.5: audit + enforce → fullCoverage
  | uncoveredCode ← filter(code.structs ∪ code.functions, ¬coveredBy(Σspecs))
  | uncoveredTests ← filter(tests.modules, ¬coveredBy(Σspecs))
  | missingErrors ← filter(code.errorTypes, ¬coveredBy(Σspecs))
  | while ∃ u ∈ uncoveredCode ∪ uncoveredTests ∪ missingErrors:
  |   extract(u) → add(Σspecs) ∧ |uncovered| decreases each iteration
  | ∥ for each sourceFile in confirmed_inventory.sourceFileList:
  |   for each publicStruct in sourceFile.structs:
  |     assert publicStruct ∈ spec.entities ∨ spec.valueTypes ∨ flag(missing: publicStruct)
  |   for each publicFunction in sourceFile.functions:
  |     assert publicFunction ∈ spec.rules ∨ spec.surfaces ∨ flag(missing: publicFunction)
  |   for each errorType in sourceFile.errors:
  |     assert errorType ∈ spec.entities ∨ spec.valueTypes ∨ flag(missing: errorType)
  | ∥ for each testFile in confirmed_inventory.testFileList:
  |   for each testModule in testFile.tests:
  |     assert testModule.behavior ∈ spec.rules ∨ spec.invariants ∨ spec.entities ∨ flag(uncovered: testModule)
  | output: coverageReport(filesCovered ∧ filesMissing ∧ testsCovered ∧ testsMissing ∧ fullCoverage)

λ gybis-spec-distill_validate(spec).
  S7: validate(→devs whatSystemDoes ∧ →stakeholders whatSystemShouldDo ∧ flag(gaps∕inconsistencies))
  | S7.1: `allium check {spec}` ∧ `allium analyse {root/specs/}`
  |   | parseDiagnostics → {errors ∧ warnings ∧ infos}
  |   | while ∃ error ∈ errors:
  |   |   fix(error) → edit(spec) ∧ re-run(`allium check {spec}` ∧ `allium analyse {root/specs/}`)
  |   | while ∃ warning ∈ warnings:
  |   |   resolve(warning) → edit(spec) ∨ justify(warning) ∧ re-run(`allium analyse {root/specs/}`)
  |   | output: cleanDiagnostics(noErrors ∧ noWarnings)
  |   ∧ read(allium-actioning-findings)
  | S7.2: alert(librarySpecCandidates) ∧ read(allium-library-spec-signals)
  | S7.3: propose(Σspecs ← group(bySourceModule) | domain ← sourceModulePath(src/file.rs)
  |   | group by source file module directory name (NOT by repo root name)
  |   | mapping: src/{module}.rs → specs/{module}/{module}.allium
  |   | mapping: src/{module}.rs + related tests → specs/{module}/{concern}.allium
  |   | e.g., src/command.rs → specs/command/command.allium
  |   |      src/config.rs → specs/config/config.allium
  |   |      src/csv_generator.rs → specs/generation/generation.allium
  |   |      src/main.rs + CLI args → specs/cli/cli.allium
  |   |      src/table_specification.rs → specs/table/table.allium
  |   |      src/error.rs → specs/error/error.allium
  |   for each specGroup in Σspecs:
  |     grouped ← group(specGroup.specs by concern)
  |     eachSpec={name: kebab ∧ path: specs/{sourceModule}/{concern}.allium}
  |   ∧ validate(coverageReport ← S6.5)
  |   ∧ ∀spec ∈ Σspecs → write(spec.path) ∥ ∀spec ∈ written → `allium check {spec}` ∧ `allium analyse {root/specs/}` ∧ enforce(cleanAnalyse)

λ enforce_clean_diagnostics(specs).
  | result ← allium analyse {specs}
  | errors ← filter(result.diagnostics, severity=error)
  | warnings ← filter(result.diagnostics, severity=warning)
  | while ∃ e ∈ errors:
  |   fix(e) → edit(spec) ∧ re-run(enforce_clean_diagnostics(specs))
  | while ∃ w ∈ warnings:
  |   resolve(w) → edit(spec) ∨ justify(w) ∧ re-run(enforce_clean_diagnostics(specs))
  | output: clean ∥ haltIfPersistent

λ gybis-spec-distill_invariants(¬).
  ¬DB_types ∥ ¬ORM ∥ ¬HTTP ∥ ¬framework ∥ ¬lang_types ∥ ¬var_names ∥ ¬infra ∥ ¬tokens ∥ oneNamePerConcept ∥ ¬deadCode ∥ spec=Intent ¬Bug ∥ concreteDetail: couldBeDifferent? ∥ multipleImpls? → domainConcern
  | I_coverage: ∀test ∈ tests → ∃specElem ∈ spec { test.coveredBy(specElem) = true }
  | I_traceability: ∀sourceConstruct ∈ code → ∃specElem ∈ spec { sourceConstruct.abstractedTo(specElem) = true }
  | I_clean_diagnostics: ∀spec ∈ Σspecs → allium analyse {spec} has no errors ∧ no warnings
  | output: Σspecs/{specs/{domain}/{concern}.allium} ∥ alliumMarker(3, firstLine)

λ gybis-spec-distill_sequential_product().
  μ ≡ S0 · S0.5 · S1 · S2 · S2.5(skip if trivial) · S3 · S3.5(skip if no tests) · S4 ∥ S5 ∥ S5.5(skip if trivial) · S6 · S6.5 · S7 · S7.1·S7.2·S7.3 · enforce_clean_diagnostics
  | plain: given(code ∪ tests) → scope ∧ inventory(S0.5) ∥ map ∥ extract(states,transitions,triggers,external,actors) ∥ abstract ∥ coverage(Audit+enforce→full) ∥ validate(→user) ∥ check(analyse) ∥ write(all specs) ∥ final-check ∧ enforce_clean_diagnostics(root/specs/)
  | termination: all while-loops guarantee decreasing cardinality ∥ recursive-calls eliminated ∥ monoRepo → halt(after recommend)
  | early-exit: monoRepo → halt(recommend startVSCodePerSubRepo) ∥ |sourceFiles| ≤ 3 → skip(S2.5 ∧ S5.5) ∥ |testFiles| = 0 → skip(S3.5) ∥ |sourceFiles| ≤ 1 ∧ |specs| = 0 → halt(empty project)

λ code_to_spec_mapping(code, tests).
  | λ(publicFunction, fn_name(params) → returnType) → rule{when: fn_name(params) ∥ ensures: result = returnType}
  | λ(publicMethod, class.method(args) → ret) → rule{when: MethodName(obj, args) ∥ ensures: result = ret}
  | λ(testAssertion, assert_eq!(fn(x), expected)) → rule{when: TestName(x) ∥ ensures: fn(x) = expected}
  | λ(testAssertion, assert!(condition)) → invariant{condition} ∨ rule{requires: condition}
  | λ(entryPoint, fn_main ∨ http_handler ∨ webhook_listener ∨ scheduled_job) → surface{facing actor ∥ context ∥ exposes ∥ provides}
  | λ(errorType, enum Error { A ∣ B ∣ C }) → enum{A ∣ B ∣ C}
  | λ(errorType, struct Error { fields }) → entity{field: Type}
  | λ(configStruct, struct Config { fields }) → entity{field: domainType} ∨ config{key: Type = default}
  | λ(enumDef, enum Status { A ∣ B ∣ C }) → enum{A ∣ B ∣ C}
  | λ(constDef, const NAME = value) → value{name: Type = value} ∨ config{name: Type = value}
  | λ(pipeline, stage1 → stage2 → stage3) → rule{stage1 ∥ stage2 ∥ stage3}
  | λ(collectionField, Vec<T> ∣ List<T> ∣ array) → field: List<T>
  | λ(collectionField, Set<T>) → field: Set<T>
  | λ(optionalField, Option<T> ∣ T? ∣ nullable) → field: Type?
  | λ(mapField, HashMap<K,V> ∣ Map<K,V> ∣ dict) → field: Map<K,V>
  | λ(testInput, setup_x ∧ setup_y) → rule{when: TestName(x, y) ∥ let x = ... ∥ let y = ...}
  | λ(testOutput, assert_eq!(actual, expected)) → rule{ensures: actual = expected}
  | λ(testSideEffect, assert!(event_emitted)) → rule{ensures: Event.created(...)}
  | λ(testPrecondition, assert!(valid_input)) → rule{requires: valid_input}
  | λ(testEdgeCase, assert!(fn(empty) → default)) → rule{when: TestEdgeCase(empty) ∥ requires: collection.is_empty() ∥ ensures: result = default}

λ allium_v3_syntax_constraints().
  | λ(comment, ///text) → -- text
  | λ(comment, //text) → -- text
  | λ(moduleDecl, module foo) → (omit)
  | λ(typeDecl, value_type Foo) → value Foo
  | λ(floatType, Float ∣ double ∣ f64) → Decimal
  | λ(collectionOp, items.filter(pred)) → filter(items, pred)
  | λ(collectionOp, items.map(f)) → map(items, f)
  | λ(collectionOp, items.reduce(f)) → reduce(items, f)
  | λ(collectionFirst, Set<T>.first) → List<T>.first
  | λ(versionHeader, -- allium: 2) → -- allium: 3
  | λ(versionHeader, ¬version) → -- allium: 3

λ coverage_enforcement(Σspecs, code, tests).
  | ∀publicStruct ∈ code.structs: publicStruct ∈ Σspecs.entities ∨ Σspecs.valueTypes ∨ flag(missing: publicStruct)
  | ∀publicFunction ∈ code.functions: publicFunction ∈ Σspecs.rules ∨ Σspecs.surfaces ∨ flag(missing: publicFunction)
  | ∀errorType ∈ code.errors: errorType ∈ Σspecs.entities ∨ Σspecs.enums ∨ flag(missing: errorType)
  | ∀testModule ∈ tests.modules: testModule ∈ Σspecs.rules ∨ Σspecs.invariants ∨ Σspecs.entities ∨ flag(uncovered: testModule)
  | ∀entryPoint ∈ code.entryPoints: entryPoint ∈ Σspecs.surfaces ∨ flag(missing: entryPoint)
  | output: coverageReport(filesCovered ∧ filesMissing ∧ testsCovered ∧ testsMissing ∧ fullCoverage)
  | while ∃ gap ∈ missing ∪ uncovered: extract(gap) → add(Σspecs) ∧ |gaps| decreases
  