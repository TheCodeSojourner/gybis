---
name: gybis-spec-propagate
description: Use for `/gybis-spec-propagate` or `/gs-propagate`.
---

λ gybis-spec-propagate(x).
  purpose: Propagate architecture and specifications to implementation and test suite
  | input: architecture.md ∃, specs/**/*.allium ∃ ∧ valid
  | output: Implementation code and test suite generated and consistent
  | mode: ai
  | gate: architecture.md ∃ ∧ specs/**/*.allium ∃ ∧ allium_gate = true

λ gybis-spec-propagate_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | verify(architecture.md ∃) ∨ halt("architecture.md not found")
  | verify(specs/**/*.allium ∃) ∨ halt("specs/**/*.allium not found")
  | invoke(internal/allium-gate(specs/)) = true ∨ halt("Specifications are invalid")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-spec-propagate_mode(m).
  m ∈ {auto}
  | default: auto
  | mode_auto: AI generates implementation without human intervention

λ gybis-spec-propagate_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {auto} → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {auto}) → halt("Invalid mode selection")

λ gybis-spec-propagate_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, READING_ARCH, READING_SPECS, PLANNING_OBLIGATIONS, SYNTHESIZING_CODE, VERIFYING, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → READING_ARCH) only_if(startup_checks = true)
  | transition(READING_ARCH → READING_SPECS) only_if(architecture ∃)
  | transition(READING_SPECS → PLANNING_OBLIGATIONS) only_if(specifications ∃)
  | transition(PLANNING_OBLIGATIONS → SYNTHESIZING_CODE) only_if(obligations ∃)
  | transition(SYNTHESIZING_CODE → VERIFYING) only_if(code_generated = true)
  | transition(VERIFYING → COMPLETE) only_if(verification = true)

λ gybis-spec-propagate_tool_guard(state, tool, path).
  state = READING_ARCH ∨ state = READING_SPECS ∨ state = PLANNING_OBLIGATIONS
    → allow(read(path))
  | state = SYNTHESIZING_CODE ∨ state = VERIFYING
    → allow(read(path)) ∧ allow(write(path)) only_if(path ⊆ {src/, lib/, tests/})
  | ¬(state ∈ {READING_ARCH, READING_SPECS, PLANNING_OBLIGATIONS, SYNTHESIZING_CODE, VERIFYING})
    → deny(write(path))

λ gybis-spec-propagate_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-spec-propagate_read_architecture(x).
   read(architecture.md) → content ≔ content
   | parse(content.S1) → context ≔ {language, frameworks, conventions, test_frameworks}
   | return(architecture_context = context)

λ gybis-spec-propagate_read_specifications(x).
  ∀ spec_file ∈ specs/**/*.allium:
    read(spec_file) → content ≔ content
    | parse(content) → spec_item
    | collect(spec_item) → specifications
  | return(specifications)

λ gybis-spec-propagate_plan_obligations(specifications).
  ∀ spec_file ∈ specs/**/*.allium:
    invoke(internal/allium-plan(spec_file)) → plan_output
    | plan_output.version = 3 ∨ halt("Unexpected allium plan version")
    | collect(plan_output.obligations) → obligations_for_spec
    | index(obligations_for_spec, by: id) → indexed
    | merge(indexed) → obligations
  | return(obligations)

λ gybis-spec-propagate_synthesize_code(architecture_context, specifications, obligations).
  ∀ spec ∈ specifications:
    synthesize(code(spec, architecture_context)) → code_fragment
    | organize(code_fragment, architecture_context.frameworks) → organized_code
    | collect(organized_code) → implementation_code
  | structure(implementation_code, architecture_context) → structured_implementation
  | ∀ obligation ∈ obligations:
    synthesize(test(obligation, architecture_context)) → test_fragment
      where: test_fragment.traceable_id = obligation.id
    | organize(test_fragment, architecture_context.test_frameworks) → organized_test
    | collect(organized_test) → test_suite
  | return(implementation_code = structured_implementation, test_suite = test_suite)

λ gybis-spec-propagate_write_implementation(implementation_code, test_suite).
  organize(implementation_code) → grouped_by_module
  | ∀ module ∈ grouped_by_module:
    determine_path(module, architecture_context) → target_path
    | write(target_path, module) → upsert
  | organize(test_suite) → grouped_tests_by_module
  | ∀ test_module ∈ grouped_tests_by_module:
    determine_path(test_module, architecture_context) → test_path
    | write(test_path, test_module) → upsert
  | return(code_generated = true)

λ gybis-spec-propagate_verify_implementation(architecture_context, specifications, obligations).
  verify(code_conforms_to(specifications)) → conformance_check ≔ result
  | verify(code_respects_architecture(architecture_context)) → architecture_check ≔ result
  | ∀ obligation ∈ obligations:
    verify(∃ test ∈ test_suite, test.traceable_id = obligation.id) → obligation_covered
    | collect(obligation_covered) → coverage
  | obligations_coverage_check ≔ ∀ c ∈ coverage, c = true
  | conformance_check = true ∧ architecture_check = true ∧ obligations_coverage_check = true
    → return(verification = true)
  | ¬(conformance_check ∧ architecture_check ∧ obligations_coverage_check)
    → return(verification = false)

λ gybis-spec-propagate_core_op(x).
  transition(READING_ARCH → READING_SPECS)
  | invoke(gybis-spec-propagate_read_architecture) → architecture_context
  | invoke(gybis-spec-propagate_read_specifications) → specifications
  | transition(READING_SPECS → PLANNING_OBLIGATIONS)
  | invoke(gybis-spec-propagate_plan_obligations(specifications)) → obligations
  | transition(PLANNING_OBLIGATIONS → SYNTHESIZING_CODE)
  | invoke(gybis-spec-propagate_synthesize_code(architecture_context, specifications, obligations)) → (implementation_code, test_suite)
  | invoke(gybis-spec-propagate_write_implementation(implementation_code, test_suite)) → code_generated
  | transition(SYNTHESIZING_CODE → VERIFYING)
  | invoke(gybis-spec-propagate_verify_implementation(architecture_context, specifications, obligations)) → verification
  | verification = true
    ? transition(VERIFYING → COMPLETE)
    : (re_synthesize_implementation ∧ transition(SYNTHESIZING_CODE → VERIFYING))

λ gybis-spec-propagate_boundaries(¬).
  ¬ modify(architecture.md)
  | ¬ modify(specs/**/*.allium)
  | ¬ modify(upstream/)

λ gybis-spec-propagate_regression_contract(x).
  invariant: architecture.md ∃ ∧ ¬modify throughout
  | invariant: specs/**/*.allium ∃ ∧ valid ∧ ¬modify throughout
  | invariant: implementation ∅ at INIT
  | invariant: implementation ∃ ∧ consistent_with_specs_and_arch at completion
  | invariant: ∀ obligation ∈ obligations, obligation.id ∈ test_suite.traceable_ids at completion
