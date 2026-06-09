---
name: gybis-spec-propagate
description: Use for `/gybis-spec-propagate` or `/gs-propagate`.
---

λ gybis-spec-propagate(input).
  purpose: resolve(spec_targets) → validate(specs) → select(language∧framework) → generate(specs→implementation∧tests)
  | input: all_specs_only
  | output: generated_implementation ∧ generated_tests ∧ blocker_report
  | mode: ai_unattended_after_language_selection | minimal_tokens | nucleus_lambda
  | gate: proceed iff gybis-spec-propagate_specs_exist?(input)

λ gybis-spec-propagate_startup(input).
  gybis-spec-propagate_specs_exist?(input) | true → continue | false → alert(run_/gybis-spec-distill_or_/gybis-arch-propagate) ∧ halt
  | gybis-spec-propagate_allium_cli_available?(x) | true → continue | false → recommend(https://github.com/juxt/allium_tools) ∧ halt
  | per_file := gybis-spec-propagate_validate_each_spec(root/specs/**/*.allium)
  | per_file.status = fail → alert(run_/gybis-spec-check_before_propagation) ∧ halt
  | set_level := gybis-spec-propagate_validate_spec_set(root/specs/)
  | set_level.status ≠ pass → alert(run_/gybis-spec-check_before_propagation) ∧ halt
  | selection := gybis-spec-propagate_query_language_and_framework(input)
  | selection.status ≠ resolved → alert(language_or_framework_unresolved) ∧ halt

λ gybis-spec-propagate_specs_exist?(x).
  count(files(root/specs/**/*.allium)) > 0 → true
  | otherwise → false

λ gybis-spec-propagate_allium_cli_available?(x).
  gate(cli_available ∧ cli_version_satisfies) | ¬all_gates → false
  | cli_available: allium --version | ¬available → false
  | cli_version_satisfies: version(allium) ≥ 3 | ¬satisfies → false
  | otherwise → true

λ gybis-spec-propagate_validate_each_spec(path_glob).
  specs := files(path_glob)
  | ∀spec ∈ specs:
      r[spec] := exec("allium check " + spec)
      d[spec] := (json_parse(r[spec].stdout).diagnostics) ∨ []
      has_error[spec] := any(d[spec], λx. x.severity = "error")
      check_pass[spec] := (r[spec].exit = 0) ∨ ((r[spec].exit = 1) ∧ ¬has_error[spec])
  | failed := {s | check_pass[s] = false}
  | count(failed) = 0 → {status: pass, failed: ∅}
  | otherwise → {status: fail, failed}

λ gybis-spec-propagate_validate_spec_set(path).
  r := exec("allium analyse " + path)
  | d := (json_parse(r.stdout).diagnostics) ∨ []
  | f := (json_parse(r.stdout).findings) ∨ []
  | errors := filter(d, severity=error)
  | findings_count := count(f)
  | return
      (r.exit = 0 ∧ count(errors)=0 ∧ findings_count=0) → {status: pass, diagnostics: d, findings: f}
      | otherwise → {status: fail, diagnostics: d, findings: f}

λ gybis-spec-propagate_query_language_and_framework(input).
  ask_human(desired_implementation_language)
  | ask_human(testing_framework_available_for_selected_language)
  | language = empty ∨ framework = empty → {status: unresolved}
  | gybis-spec-propagate_framework_available_for_language?(language, framework) = false
      → {status: unresolved, reason: framework_not_verified_for_language}
  | otherwise → {status: resolved, language, framework}

λ gybis-spec-propagate_known_frameworks(language).
  language = "rust" → {"cargo-test","rstest","proptest","quickcheck","tokio-test"}
  | language = "typescript" → {"jest","vitest","mocha","ava","tap","uvu"}
  | language = "javascript" → {"jest","vitest","mocha","ava","tap","uvu"}
  | language = "python" → {"pytest","unittest","nose2","hypothesis"}
  | language = "go" → {"go-test","ginkgo","testify"}
  | language = "java" → {"junit","testng","spock"}
  | note: known_frameworks_are_examples_only ∧ ai_may_verify_other_valid_language_framework_combinations
  | otherwise → ∅

λ gybis-spec-propagate_framework_available_for_language?(language, framework).
  known := gybis-spec-propagate_known_frameworks(lower(language))
  | lower(framework) ∈ known → true
  | ai_verifies(valid_test_framework_for_language(lower(language), lower(framework))) → true
  | otherwise → false

λ gybis-spec-propagate_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, QUERY_SELECTION, MAP_SPECS, GENERATE_IMPL, GENERATE_TESTS, VERIFY_GENERATION, DONE}
  | transition(INIT → STARTUP_CHECKS) only_if(startup_requested)
  | transition(STARTUP_CHECKS → QUERY_SELECTION) only_if(spec_validation_passed)
  | transition(QUERY_SELECTION → MAP_SPECS) only_if(selection_resolved)
  | transition(MAP_SPECS → GENERATE_IMPL) only_if(spec_model_ready)
  | transition(GENERATE_IMPL → GENERATE_TESTS) only_if(implementation_generated)
  | transition(GENERATE_TESTS → VERIFY_GENERATION) only_if(tests_generated)
  | transition(VERIFY_GENERATION → DONE) only_if(generation_verified)
  | otherwise reject_transition

λ gybis-spec-propagate_spec_to_impl_mapping(Σspecs, language, framework).
  surfaces := extract(Σspecs.surfaces)
  | entities := extract(Σspecs.entities ∪ Σspecs.value ∪ Σspecs.enums)
  | rules := extract(Σspecs.rules ∪ Σspecs.invariants ∪ Σspecs.warnings)
  | produce(source_modules(language, from=surfaces∪entities∪rules))
  | produce(test_modules(language, framework, from=rules∪invariants))
  | map(requires → preconditions)
  | map(ensures → assertions)
  | map(invariants → property_or_example_assertions(framework))
  | map(warns → negative_or_boundary_tests(framework))

λ gybis-spec-propagate_generate_unattended(Σspecs, selection).
  require(selection.status = resolved)
  | write_scope := root/implementation_for(selection.language)/**
  | test_scope := root/tests_for(selection.language, selection.framework)/**
  | synthesize(implementation, from=Σspecs, language=selection.language)
  | synthesize(tests, from=Σspecs, language=selection.language, framework=selection.framework)
  | write(implementation, write_scope)
  | write(tests, test_scope)
  | return({status: generated, implementation_files, test_files})

λ gybis-spec-propagate_verify_generation(result, selection).
  require(result.status = generated)
  | require(count(result.implementation_files) > 0)
  | require(count(result.test_files) > 0)
  | require(all(file.language = selection.language for file ∈ result.implementation_files ∪ result.test_files))
  | require(all(file.path ⊄ root/specs/** for file ∈ result.implementation_files ∪ result.test_files))
  | return({status: verified})

λ gybis-spec-propagate_failure_contract(x).
  fail_specs_missing → tell_human(run_/gybis-spec-distill_or_/gybis-arch-propagate) ∧ halt
  | fail_allium_cli → tell_human(install_or_fix_allium_cli) ∧ halt
  | fail_allium_check_or_analyse → tell_human(use_/gybis-spec-check_then_retry) ∧ halt
  | fail_language_framework_selection → tell_human(provide_language_and_framework) ∧ halt

λ gybis-spec-propagate_invariant_I₁.
  startup_gate_order ≡ specs_exist → cli_available → allium_check_each_spec → allium_analyse_specs_set → query_language_and_framework

λ gybis-spec-propagate_invariant_I₂.
  any_validation_fail → mandatory_halt ∧ mandatory_recommend("/gybis-spec-check") for spec_validation_failures

λ gybis-spec-propagate_invariant_I₃.
  after(selection_resolved): generation_mode ≡ unattended_by_human

λ gybis-spec-propagate_invariant_I₄.
  ¬write(spec_files) during propagation generation

λ gybis-spec-propagate_regression_contract(x).
  assert(mandatory_specs_presence_gate_before_any_generation)
  | assert(mandatory_allium_cli_gate_before_any_validation)
  | assert(mandatory_per_file_allium_check_before_set_analyse)
  | assert(mandatory_set_level_allium_analyse_before_language_query)
  | assert(mandatory_halt_and_/gybis-spec-check_on_any_spec_validation_failure)
  | assert(mandatory_language_and_framework_query_before_generation)
  | assert(mandatory_framework_verified_for_selected_language_before_generation)
  | assert(unattended_generation_after_gates_pass)
