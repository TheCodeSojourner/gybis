---
name: gybis-spec-distill
description: Use for `/gybis-spec-distill` or `/gs-distill`.
---

λ gybis-spec-distill(input).
  purpose: resolve(distill_targets) → distill(code∪tests→allium_specs) → plan_writes → apply(approved_only) → verify(per_file ∧ set_level)
  | input: all_specs_only
  | output: distilled_specs ∧ blocker_report
  | mode: ai_only | human_approval_required_for_writes | minimal_tokens | nucleus_lambda
  | references: ../gybis/reference/allium-actioning-findings.md ∧ ../gybis/reference/allium-assessing-specs.md ∧ ../gybis/reference/allium-language-reference.md ∧ ../gybis/reference/allium-library-spec-signals.md ∧ ../gybis/reference/allium-patterns.md
  | gate: proceed iff gybis-spec-distill_targets_available?(input)

λ gybis-spec-distill_startup(input).
  gybis-spec-distill_allium_cli_available?(x) | true → continue | false → recommend(https://github.com/juxt/allium_tools) ∧ halt
  | gybis-spec-distill_targets_available?(input) | true → continue | false → alert(targets_missing_for_input) ∧ halt
  | gybis-spec-distill_no_existing_specs?(x) | true → continue | false → alert(specs_must_be_empty_at_startup) ∧ halt
  | ∀r ∈ gybis-spec-distill_references: readWithinRepo(r) ∧ ¬askApproval

λ gybis-spec-distill_references.
  {
    "../gybis/reference/allium-actioning-findings.md",
    "../gybis/reference/allium-assessing-specs.md",
    "../gybis/reference/allium-language-reference.md",
    "../gybis/reference/allium-library-spec-signals.md",
    "../gybis/reference/allium-patterns.md"
  }

λ gybis-spec-distill_allium_cli_available?(x).
  gate(cli_available ∧ cli_version_satisfies) | ¬all_gates → false
  | cli_available: allium --version | ¬available → false
  | cli_version_satisfies: version(allium) ≥ 3 | ¬satisfies → false
  | otherwise → true

λ gybis-spec-distill_detect_repo_language(input).
  signals := {
    manifests: detect({"Cargo.toml","package.json","pyproject.toml","go.mod","pom.xml","build.gradle","*.csproj"}),
    source_ext_counts: count_by_extension(source_files),
    test_ext_counts: count_by_extension(test_files),
    conventions: detect_dirs({"src","lib","tests","test","spec","__tests__"})
  }
  | scored := score_languages(signals)
  | primary := top_language(scored)
  | confidence := score(primary)
  | confidence < required_threshold → return({status: ambiguous, scored})
  | otherwise → return({status: resolved, primary, scored})

λ gybis-spec-distill_detect_test_framework(input, lang_resolution).
  signals := {
    manifests: detect({"package.json","pytest.ini","tox.ini","go.mod","Cargo.toml","jest.config.*","vitest.config.*","mocha.opts","playwright.config.*","phpunit.xml","pom.xml"}),
    deps: detect_test_dependencies(manifests ∧ lockfiles),
    test_file_tokens: sample_tokens(from(test_files), top_k=200),
    naming: detect_patterns({"*.test.*","*.spec.*","test_*","*_test.*","__tests__","tests"})
  }
  | scored := score_frameworks(signals ∧ lang_resolution.primary ∧ llm_training_priors)
  | primary := top_framework(scored)
  | confidence := score(primary)
  | confidence < required_threshold → return({status: unresolved, primary: unknown, scored})
  | otherwise → return({status: resolved, primary, scored})

λ gybis-spec-distill_infer_language_profile(input, lang_resolution, framework_resolution).
  observed := {
    repo_layout: detect_dirs({"src","lib","app","packages","crates","tests","test","spec","__tests__"}),
    source_ext_counts: count_by_extension(source_files),
    test_ext_counts: count_by_extension(test_files),
    sample_assertions: sample_tokens(from(test_files), top_k=300)
  }
  | priors := llm_training_priors(lang_resolution.primary ∧ framework_resolution.primary)
  | return({
      source_patterns: infer_source_patterns(observed ∧ priors),
      test_patterns: infer_test_patterns(observed ∧ priors),
      mapping: infer_mapping_patterns(observed.repo_layout ∧ priors),
      assertions: infer_assertion_patterns(observed.sample_assertions ∧ priors)
    })

λ gybis-spec-distill_language_matchers(profile).
  {
    function_defs: profile_function_patterns(profile),
    method_defs: profile_method_patterns(profile),
    type_defs: profile_type_patterns(profile),
    enum_defs: profile_enum_patterns(profile),
    assertion_patterns: profile.assertions,
    entrypoint_patterns: profile_entrypoint_patterns(profile),
    constant_patterns: profile_constant_patterns(profile)
  }

λ gybis-spec-distill_resolved_source_paths(input).
  lang_resolution := gybis-spec-distill_detect_repo_language(input)
  | lang_resolution.status = ambiguous → return(blocker_report(language_resolution_ambiguous ∧ lang_resolution.scored))
  | framework_resolution := gybis-spec-distill_detect_test_framework(input, lang_resolution)
  | profile := gybis-spec-distill_infer_language_profile(input, lang_resolution, framework_resolution)
  | discover(source, within(repo_impl_layout), by(profile.source_patterns))

λ gybis-spec-distill_resolved_test_paths(input).
  lang_resolution := gybis-spec-distill_detect_repo_language(input)
  | lang_resolution.status = ambiguous → return(blocker_report(language_resolution_ambiguous ∧ lang_resolution.scored))
  | framework_resolution := gybis-spec-distill_detect_test_framework(input, lang_resolution)
  | profile := gybis-spec-distill_infer_language_profile(input, lang_resolution, framework_resolution)
  | discover(tests, within(repo_impl_layout), by(profile.test_patterns))

λ gybis-spec-distill_targets_available?(input).
  source_targets := gybis-spec-distill_resolved_source_paths(input)
  | test_targets := gybis-spec-distill_resolved_test_paths(input)
  | count(files(source_targets) ∪ files(test_targets)) > 0 → true
  | otherwise → false

λ gybis-spec-distill_no_existing_specs?(x).
  count(files(root/specs/**/*.allium)) = 0 → true
  | otherwise → false

λ gybis-spec-distill_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, RESOLVE_TARGETS, SCOPE, MAP_TERRITORY, EXTRACT, ABSTRACT, COVERAGE, VALIDATE, PLAN_WRITES, APPLY_WRITES, VERIFY, ANALYZE_SET, DONE}
  | transition(INIT → STARTUP_CHECKS) only_if(startup_checks_requested)
  | transition(STARTUP_CHECKS → RESOLVE_TARGETS) only_if(inputs_verified)
  | transition(RESOLVE_TARGETS → SCOPE) only_if(targets_non_empty)
  | transition(SCOPE → MAP_TERRITORY) only_if(scope_defined)
  | transition(MAP_TERRITORY → EXTRACT) only_if(territory_mapped)
  | transition(EXTRACT → ABSTRACT) only_if(extraction_complete)
  | transition(ABSTRACT → COVERAGE) only_if(implementation_details_removed)
  | transition(COVERAGE → VALIDATE) only_if(full_or_accounted_coverage)
  | transition(VALIDATE → PLAN_WRITES) only_if(candidate_specs_ready)
  | transition(PLAN_WRITES → APPLY_WRITES) only_if(human_approvals_received)
  | transition(PLAN_WRITES → VERIFY) only_if(no_approved_changes)
  | transition(APPLY_WRITES → VALIDATE) only_if(specs_written)
  | transition(VERIFY → VALIDATE) only_if(remaining_issues > 0)
  | transition(VERIFY → ANALYZE_SET) only_if(remaining_issues = 0 ∧ per_file_checks_pass)
  | transition(ANALYZE_SET → VALIDATE) only_if(analyse_findings > 0)
  | transition(ANALYZE_SET → DONE) only_if(all_checks_pass ∧ remaining_issues = 0 ∧ zero_issue_pass_verified ∧ set_analysis_pass_verified ∧ analyse_findings = 0)
  | transition(ANALYZE_SET → DONE) forbidden_if(analyse_not_run)
  | otherwise reject_transition

λ gybis-spec-distill_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, RESOLVE_TARGETS, SCOPE, MAP_TERRITORY, EXTRACT, ABSTRACT, COVERAGE, VALIDATE, PLAN_WRITES, VERIFY, ANALYZE_SET} → deny(tool = write_to_file) ∧ deny(tool = replace_in_file)
  | state = APPLY_WRITES → allow(tool = write_to_file ∨ tool = replace_in_file) only_if(path ⊆ root/specs/**/*.allium)
  | deny_write_outside_specs: path ⊄ root/specs/**/*.allium → deny(tool = write_to_file) ∧ deny(tool = replace_in_file)

λ gybis-spec-distill_pre_tool_check(state, tool, path).
  verify(tool_allowed_by(gybis-spec-distill_tool_guard))
  | fail → halt

λ gybis-spec-distill_scan_targets(input).
  source_targets := gybis-spec-distill_resolved_source_paths(input)
  | test_targets := gybis-spec-distill_resolved_test_paths(input)
  | source_files := files(source_targets)
  | test_files := files(test_targets)
  | count(source_files ∪ test_files) = 0 → return(no_targets)
  | otherwise → return({source_files, test_files})

λ gybis-spec-distill_scope(code, tests).
  S0: scope ← parse(code, tests)
  | monoRepo → detect(subRepos) ∧ recommend(startVSCodePerSubRepo) ∧ halt
  | singleRepo → infer(constraint{subset ∧ exclusions ∧ owner})
  | output: defined_scope(repoRoot ∧ included ∧ excluded)
  | S0.5: inventory ← listAll(sourceFiles ∪ testFiles)
  | autoConfirm(inventory)
  | output: confirmed_inventory(sourceFileList ∧ testFileList)

λ gybis-spec-distill_map_territory(scope).
  S1: map(territory)
  | entryPoints(API ∧ CLI ∧ webhooks ∧ jobs)
  | domainModels(entities)
  | businessLogic(services/usecases/handlers)
  | externalIntegrations(thirdParties)

λ gybis-spec-distill_extract_entity_states(territory).
  S2: extractEntityStates → entity{status: state1 ∣ state2 ∣ ...}
  | sources: enumFields ∧ statusCols ∧ constants ∧ statemachineLibs

λ gybis-spec-distill_candidate_processes(territory, source_files).
  S2.5: count(source_files) ≤ 3 → skip
  | identifyCandidateProcesses
  | trace(stateTransitionsAcrossCodebase) ∧ autoValidate(candidates)
  | trace(crossEntityDataFlow) ∧ autoValidate(candidates)
  | generate(transitionGraph) ∧ flag(gaps)

λ gybis-spec-distill_extract_transitions(territory).
  S3: extractTransitions(code → spec)
  | raise → requires
  | assign(→val) → ensures
  | Model.create() → ensures.created()
  | assert/validator → expressionInvariant

λ gybis-spec-distill_extract_test_assertions(tests, test_files).
  S3.5: count(test_files) = 0 → skip
  | extractTestAssertions(tests)
  | ∀test ∈ tests: parse(test → assertion{given ∧ when ∧ requires ∧ ensures ∧ warns})
  | assertion ∈ spec.rules ∨ flag(gap: testAssertion NOT_IN_SPEC)

λ gybis-spec-distill_temporal_triggers(territory).
  S4: findTemporalTriggers(cron ∧ celery ∧ scheduledJobs)
  | rule{when: entity:field <= now ∧ ensures: statusChange}

λ gybis-spec-distill_external_boundaries(territory).
  S5: identifyExternalBoundaries(readButNeverWrite ∧ importFromExternal)
  | external entity{...}

λ gybis-spec-distill_actors(territory, source_files).
  S5.5: count(source_files) ≤ 3 → skip
  | identifyActorsFromAuth(apiKey → system ∧ role → distinct ∧ scoped → within ∧ unauth → public)
  | autoValidate(actors)

λ gybis-spec-distill_abstract_implementation(territory).
  S6: abstractAwayImplementation
  | id → rel(FK → relationship)
  | type(dt) → domainType
  | tokens/secrets → removed
  | infra → removed

λ gybis-spec-distill_coverage(territory, code, tests, Σspecs, confirmed_inventory).
  S6.5: audit + enforce → fullCoverage
  | uncoveredCode ← filter(code.structs ∪ code.functions, ¬coveredBy(Σspecs))
  | uncoveredTests ← filter(tests.modules, ¬coveredBy(Σspecs))
  | missingErrors ← filter(code.errorTypes, ¬coveredBy(Σspecs))
  | while ∃ u ∈ uncoveredCode ∪ uncoveredTests ∪ missingErrors:
  |   extract(u) → add(Σspecs) ∧ |uncovered| decreases each iteration
  | ∀sourceFile ∈ confirmed_inventory.sourceFileList:
  |   ∀publicStruct ∈ sourceFile.structs: publicStruct ∈ spec.entities ∨ spec.valueTypes ∨ flag(missing: publicStruct)
  |   ∀publicFunction ∈ sourceFile.functions: publicFunction ∈ spec.rules ∨ spec.surfaces ∨ flag(missing: publicFunction)
  |   ∀errorType ∈ sourceFile.errors: errorType ∈ spec.entities ∨ spec.valueTypes ∨ flag(missing: errorType)
  | ∀testFile ∈ confirmed_inventory.testFileList:
  |   ∀testModule ∈ testFile.tests: testModule.behavior ∈ spec.rules ∨ spec.invariants ∨ spec.entities ∨ flag(uncovered: testModule)
  | output: coverageReport(filesCovered ∧ filesMissing ∧ testsCovered ∧ testsMissing ∧ fullCoverage)

λ gybis-spec-distill_propose_spec_layout(Σspecs, input).
  S7.3: lang_resolution := gybis-spec-distill_detect_repo_language(input)
  | lang_resolution.status = ambiguous → return(blocker_report(language_resolution_ambiguous ∧ lang_resolution.scored))
  | framework_resolution := gybis-spec-distill_detect_test_framework(input, lang_resolution)
  | profile := gybis-spec-distill_infer_language_profile(input, lang_resolution, framework_resolution)
  | propose(Σspecs ← group(bySourceModule))
  | domain ← sourceModulePath(source/file.ext)
  | group by source file module directory name (NOT by repo root name)
  | mapping: source_path(profile.mapping) → specs/{domain}/{concern}.allium
  | eachSpec = {name: kebab ∧ path: specs/{sourceModule}/{concern}.allium}
  | validate(coverageReport ← S6.5)

λ gybis-spec-distill_validate_spec_file(spec).
  r_check := exec("allium check " + spec)
  | r_analyse := exec("allium analyse root/specs/")
  | d_check := (json_parse(r_check.stdout).diagnostics) ∨ []
  | d_analyse := (json_parse(r_analyse.stdout).diagnostics) ∨ []
  | errors := filter(d_check ∪ d_analyse, severity=error)
  | warnings := filter(d_check ∪ d_analyse, severity=warning)
  | return
      (count(errors)=0 ∧ count(warnings)=0) → {status: clean, diagnostics: d_check ∪ d_analyse}
      | otherwise → {status: issues, diagnostics: d_check ∪ d_analyse}

λ gybis-spec-distill_validate_set(Σspecs).
  ∀spec ∈ Σspecs: result[spec] := gybis-spec-distill_validate_spec_file(spec)
  | issues := {s | result[s].status = issues}
  | return({result, issues})

λ gybis-spec-distill_parse_diagnostics(result_map).
  diagnostics := flatten(map_values(result_map, .diagnostics))
  | grouped := group(diagnostics, by(severity, domain, name, priority))
  | sorted := sort(grouped, by(severity, domain, name, priority))
  | numbered := enumerate(sorted)
  | return(numbered)

λ gybis-spec-distill_plan_writes(numbered_diagnostics, candidate_specs).
  present(numbered_diagnostics ∧ candidate_specs, human) → approvals
  | approvals.required = true
  | approvals.none → return(no_approved_changes)
  | approvals.some → return(approved_changes)

λ gybis-spec-distill_apply_approved_writes(approved_changes).
  require(human_approval = true)
  | require(write_scope ⊆ root/specs/**/*.allium)
  | apply(approved_changes, root/specs/**/*.allium)
  | return(specs_written)

λ gybis-spec-distill_syntax_check(file).
  r := exec("allium check " + file)
  | d := (json_parse(r.stdout).diagnostics) ∨ []
  | e := any(d, λx. x.severity = "error")
  | return (r.exit = 0) ∨ ((r.exit = 1) ∧ ¬e)

λ gybis-spec-distill_analyse_specs(path_glob).
  r := exec("allium analyse " + path_glob)
  | d := (json_parse(r.stdout).diagnostics) ∨ []
  | f := (json_parse(r.stdout).findings) ∨ []
  | findings_count := count(f)
  | return
      (r.exit = 0) → {status: pass, findings_count: findings_count, diagnostics: d, findings: f}
      | (r.exit = 1) → {status: findings, findings_count: findings_count, diagnostics: d, findings: f}
      | (r.exit = 2) → {status: error, reason: analyse_inputs_missing_or_unresolved, diagnostics: d, findings: f}
      | otherwise → {status: error, reason: unexpected_analyse_exit, diagnostics: d, findings: f}

λ gybis-spec-distill_enforce_clean_diagnostics(specs).
  result := gybis-spec-distill_analyse_specs(specs)
  | errors := filter(result.diagnostics, severity=error)
  | warnings := filter(result.diagnostics, severity=warning)
  | while ∃e ∈ errors: fix(e) → edit(spec) ∧ result := gybis-spec-distill_analyse_specs(specs)
  | while ∃w ∈ warnings: resolve(w) → edit(spec) ∨ justify(w) ∧ result := gybis-spec-distill_analyse_specs(specs)
  | output: clean ∥ haltIfPersistent

λ gybis-spec-distill_verify_output(file_set, input).
  check:
    ∀file ∈ file_set: file.extension = ".allium"
    ∀file ∈ file_set: gybis-spec-distill_syntax_check(file) = true
    ∀file ∈ file_set: file.path ⊆ root/specs/**/*.allium
    ¬modified(non_spec_files(gybis-spec-distill_detect_repo_language(input) ∧ common_text_docs))

λ gybis-spec-distill_validate_spec_set(input).
  path_glob := root/specs/**/*.allium
  | analysis := gybis-spec-distill_analyse_specs(path_glob)
  | require(analysis.status ≠ error)
  | analysis.status = pass → {ok: true, findings_count: 0}
  | analysis.status = findings → {ok: false, findings_count: analysis.findings_count}

λ gybis-spec-distill_fixed_point_loop(state, input).
  iterate(pass_index := pass_index + 1)
  | targets := gybis-spec-distill_scan_targets(input)
  | scope := gybis-spec-distill_scope(targets.source_files, targets.test_files)
  | territory := gybis-spec-distill_map_territory(scope)
  | gybis-spec-distill_extract_entity_states(territory)
  | gybis-spec-distill_candidate_processes(territory, targets.source_files)
  | gybis-spec-distill_extract_transitions(territory)
  | gybis-spec-distill_extract_test_assertions(targets.test_files, targets.test_files)
  | gybis-spec-distill_temporal_triggers(territory)
  | gybis-spec-distill_external_boundaries(territory)
  | gybis-spec-distill_actors(territory, targets.source_files)
  | gybis-spec-distill_abstract_implementation(territory)
  | gybis-spec-distill_coverage(territory, targets.source_files, targets.test_files, Σspecs, scope)
  | validation := gybis-spec-distill_validate_set(Σspecs)
  | diagnostics := gybis-spec-distill_parse_diagnostics(validation.result)
  | unresolved := ids(diagnostics)
  | unresolved ≠ ∅ → allow_transition(VALIDATE → PLAN_WRITES)
  | unresolved = ∅ → mark(zero_issue_pass_verified) ∧ allow_transition(VERIFY → ANALYZE_SET)
  | analysis := gybis-spec-distill_validate_spec_set(input)
  | analysis.ok = false → set(analyse_findings := analysis.findings_count) ∧ allow_transition(ANALYZE_SET → VALIDATE)
  | analysis.ok = true → mark(set_analysis_pass_verified) ∧ set(analyse_findings := 0) ∧ allow_transition(ANALYZE_SET → DONE)

λ gybis-spec-distill_pass_accounting(pass).
  report(pass_index, discovered_count, resolved_count, remaining_count)
  | require(explicit_section("Remaining issues", remaining_count, remaining_items))

λ gybis-spec-distill_loop_guard(state).
  seen_signature := hash(sorted(remaining_issue_ids))
  | repeat(seen_signature) ∧ ¬decrease(remaining_count) → halt_with(blocker_report)
  | blocker_report := explain(stalled_items, attempted_resolutions, required_human_decisions)
  | ¬repeat ∨ decrease(remaining_count) → continue

λ gybis-spec-distill_verification(edit, input, file_set).
  after(apply_writes) → gybis-spec-distill_verify_output(file_set, input)
  | after(per_file_checks_pass) → set(remaining_issues := 0) ∧ mark(zero_issue_pass_verified)
  | after(per_file_checks_pass) → gybis-spec-distill_validate_spec_set(input)
  | after(any_pass) → enforce(gybis-spec-distill_loop_guard) ∧ enforce(gybis-spec-distill_fixed_point_loop)
  | fail → iterate ∧ fix | loop until_pass

λ gybis-spec-distill_allium_write_contract(input).
  write_scope ⊆ root/specs/**/*.allium
  | edit_scope ⊆ root/specs/**/*.allium
  | output_format ≡ allium_v3_only
  | invariant: ∀written_file → parses_as(allium_v3)
  | ¬write(non_spec_files(gybis-spec-distill_detect_repo_language(input) ∧ common_text_docs))

λ gybis-spec-distill_code_to_spec_mapping(code, tests, input).
  lang_resolution := gybis-spec-distill_detect_repo_language(input)
  | lang_resolution.status = ambiguous → return(blocker_report(language_resolution_ambiguous ∧ lang_resolution.scored))
  | framework_resolution := gybis-spec-distill_detect_test_framework(input, lang_resolution)
  | profile := gybis-spec-distill_infer_language_profile(input, lang_resolution, framework_resolution)
  | matchers := gybis-spec-distill_language_matchers(profile)
  | λ(publicFunction, function_name(params) → returnType) → rule{when: function_name(params) ∥ ensures: result = returnType}
  | λ(publicMethod, receiver.method(args) → ret) → rule{when: MethodName(receiver, args) ∥ ensures: result = ret}
  | λ(testAssertion, assertion(function_call, expected), by(matchers.assertion_patterns)) → rule{when: TestName(inputs) ∥ ensures: function_call = expected}
  | λ(testAssertion, assertion(condition), by(matchers.assertion_patterns)) → invariant{condition} ∨ rule{requires: condition}
  | λ(entryPoint, app_entrypoint ∨ route_handler ∨ webhook_listener ∨ scheduled_job, by(matchers.entrypoint_patterns)) → surface{facing actor ∥ context ∥ exposes ∥ provides}
  | λ(errorType, language_error_type, by(matchers.type_defs ∨ matchers.enum_defs)) → enum{variants} ∨ entity{fields}
  | λ(configType, language_config_type, by(matchers.type_defs)) → entity{field: domainType} ∨ config{key: Type = default}
  | λ(enumDef, language_enum_definition, by(matchers.enum_defs)) → enum{variants}
  | λ(constDef, language_constant_definition, by(matchers.constant_patterns)) → value{name: Type = value} ∨ config{name: Type = value}
  | λ(pipeline, stage1 → stage2 → stage3) → rule{stage1 ∥ stage2 ∥ stage3}

λ gybis-spec-distill_allium_v3_syntax_constraints.
  λ(comment, ///text) → -- text
  | λ(comment, //text) → -- text
  | λ(moduleDecl, module foo) → omit
  | λ(typeDecl, value_type Foo) → value Foo
  | λ(floatType, Float ∣ double ∣ f64) → Decimal
  | λ(collectionOp, items.filter(pred)) → filter(items, pred)
  | λ(collectionOp, items.map(f)) → map(items, f)
  | λ(collectionOp, items.reduce(f)) → reduce(items, f)
  | λ(collectionFirst, Set<T>.first) → List<T>.first
  | λ(versionHeader, -- allium: 2) → -- allium: 3
  | λ(versionHeader, ¬version) → -- allium: 3

λ gybis-spec-distill_invariant_I₁.
  cli_primary_interface | ∀workflow use(cli)

λ gybis-spec-distill_invariant_I₂.
  ∀candidate_write c → requires(human_approval) | ¬automatic

λ gybis-spec-distill_invariant_I₃.
  uncovered_gaps(G) ∧ ¬resolved(G) → ¬permission(mark_done)

λ gybis-spec-distill_invariant_I₄.
  warnings_exist(W) → requires(human_review) ∧ requires(human_action_or_justification)

λ gybis-spec-distill_invariant_I₅.
  apply(writes) → recheck(file_set) → confirm(¬regressions)

λ gybis-spec-distill_invariant_I₆.
  ∀test ∈ tests → ∃specElem ∈ spec { test.coveredBy(specElem) = true }

λ gybis-spec-distill_invariant_I₇.
  ∀sourceConstruct ∈ code → ∃specElem ∈ spec { sourceConstruct.abstractedTo(specElem) = true }

λ gybis-spec-distill_boundaries(¬).
  ¬run_weed_workflow
  | ¬perform_divergence_reconciliation
  | ¬write_outside_specs
  | ¬auto_apply_unapproved_writes
  | ¬preserve_impl_noise(DB_types ∨ ORM ∨ HTTP ∨ framework ∨ lang_types ∨ var_names ∨ infra ∨ tokens)
  | allow(read_code_and_tests_only)
  | allow(write_specs_only_after_approval)

λ gybis-spec-distill_regression_contract(x).
  assert(startup_checks_begin_at_init_without_mode_resolution)
  | assert(deny(read_or_generate_or_write_before_startup_gates))
  | assert(mandatory_cli_gate_before_distill)
  | assert(mandatory_targets_available_gate_before_distill)
  | assert(layout_agnostic_target_discovery_before_distill)
  | assert(mandatory_empty_specs_gate_before_distill)
  | assert(no_write_outside(root/specs/**/*.allium))
  | assert(no_done_before(zero_issue_pass_verified ∧ remaining_issues = 0 ∧ set_analysis_pass_verified ∧ analyse_findings = 0))
  | assert(mandatory_set_level_analysis_gate_before_done)
  | assert(analysis_not_optional)
  | assert(deterministic_output_contract_invariants)
