---
name: gybis-spec-distill
description: Use for `/gybis-spec-distill` or `/gs-distill`.
---

λ gybis-spec-distill(x).
  purpose: distill Allium specifications from existing implementation
  | input: implementation source code (any language)
  | output: specs/**/*.allium files valid per allium-gate
  | mode: ai
  | gate: implementation ∃ ∧ specs/**/*.allium ¬∃

λ gybis-spec-distill_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | read(internal/reference/allium-language-reference.md) → language_ref
  | read(internal/reference/allium-patterns.md) → patterns_ref
  | read(internal/reference/allium-constructs.md) → constructs_registry
  | precondition: implementation ∃ ∧ specs/ ¬∃
  | scan(codebase) → files_found ∨ halt("no implementation found")

λ gybis-spec-distill_mode(m).
  valid_modes: {ai}
  | default: ai
  | rationale: distillation from code is deterministic analysis, not interactive

λ gybis-spec-distill_mode_gate(state, mode).
  state = INIT ∧ mode = ai → transition(STARTUP_CHECKS)
  | precondition_holds: mode ∈ valid_modes

λ gybis-spec-distill_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, READING_CODE, ANALYSING, SYNTHESIZING_SPECS, VALIDATING, REFINING, COMPLETE}
  | transition(INIT, startup) → STARTUP_CHECKS
  | transition(STARTUP_CHECKS, verify_ok) → READING_CODE
  | transition(STARTUP_CHECKS, verify_fail) → HALTED
  | transition(READING_CODE, code_read) → ANALYSING
  | transition(ANALYSING, analysis_complete) → SYNTHESIZING_SPECS
  | transition(SYNTHESIZING_SPECS, specs_written) → VALIDATING
  | transition(VALIDATING, all_valid) → COMPLETE
  | transition(VALIDATING, errors_found) → REFINING
  | transition(REFINING, refinement_complete) → VALIDATING (loop_back)

λ gybis-spec-distill_tool_guard(state, tool, path).
  read_allowed: ∀state ∈ {READING_CODE, ANALYSING, VALIDATING}
  | write_allowed: state ∈ {SYNTHESIZING_SPECS, REFINING} ∧ path ⊆ specs/
  | deny_write: state ∉ {SYNTHESIZING_SPECS, REFINING} ∨ path ⊄ specs/
  | constraint: ¬mutate(implementation) ∨ ¬mutate(existing_files_outside_specs)

λ gybis-spec-distill_pre_tool_check(state, tool, path).
  enforce(tool_guard(state, tool, path)) → permit(tool) ∨ halt("tool not permitted in this state")

λ gybis-spec-distill_read_code(x).
  action: read_all_implementation_files
  | step1: recursively_list(codebase) → files
  | step2: ∀file ∈ files: read(file) → code_content
  | step3: classify(file_type) → language_specific_handling
  | output: {file_path → source_code}
  | constraint: read-only access
  | precondition: implementation ∃

λ gybis-spec-distill_analyse_code(code_content).
  action: extract_patterns_and_contracts
  | step1: parse(code) → abstract_syntax_tree
  | step2: extract(types, functions, classes, modules) → entities
  | step3: identify(contracts, invariants, preconditions, postconditions) → behavioral_specs
  | step4: recognize(patterns, protocols, relationships) → architectural_patterns
  | step5: infer(test_cases, edge_cases, error_handling) → test_obligations
  | step6: classify(source_patterns) → allium_construct_map  -- see distill_construct_recognition
  | output: {entities → contracts, patterns → implications, obligations → test_cases, source_pattern → allium_construct}
  | constraint: read-only operation

λ gybis-spec-distill_construct_recognition(source_pattern).
  source_pattern matches Cues entry → construct ≔ constructs_registry[entry]
  | classification_rules:
      relationship_must_reference_this_via_with
      ∧ filter_must_use_where_without_this
      ∧ dot_methods_restricted_to({count, any, all, first, last, unique, add, remove})
      ∧ free_standing_call_for_all_other_collection_or_scalar_helpers
      ∧ expressible_invariants_only_when_property_is_single_point_in_time
      ∧ accretion_preferred_over_breaking_changes
  | fallback: ¬recognised_pattern → standard_entity ∧ rules

λ gybis-spec-distill_synthesize_specs(code_content, analysis).
  action: synthesize_allium_specifications
  | step1: map(entities) → .allium declarations
  | step2: encode(behavioral_specs) → allium preconditions, postconditions, invariants
  | step3: formalize(patterns) → allium composition rules
  | step4: ∀ pattern ∈ analysis.source_pattern → allium_construct:
    emit(pattern.allium_construct) → spec_fragment  -- per construct_recognition table
  | step5: generate(specs_directory_structure) → specs/**/*.allium files
  | output: {spec_file_path → allium_content}

λ gybis-spec-distill_write_specs(spec_content).
  action: write_specification_files
  | step1: ensure_specs_directory_exists(specs/)
  | step2: ∀spec ∈ spec_content: write(specs/{spec_name}.allium, spec)
  | constraint: write_allowed by tool_guard
  | precondition: state ∈ {SYNTHESIZING_SPECS, REFINING}
  | output: specs/**/*.allium files written

λ gybis-spec-distill_validate_specs(x).
  action: validate_generated_specifications
  | step1: invoke(internal/allium-check, ∀spec ∈ specs/**/*.allium)
  | step2: invoke(internal/allium-gate, specs/) → boolean_verdict
  | step3: if gate_fails: collect(errors) → diagnostic_report
  | output: validation_result ∈ {pass, fail_with_diagnostics}
  | constraint: read-only validation

λ gybis-spec-distill_verification(edit).
  action: verify_generated_specifications
  | check1: ∀spec ∈ specs/**/*.allium: file_exists(spec) = true
  | check2: ∀spec ∈ specs/**/*.allium: syntax_valid(allium) = true
  | check3: allium-gate(specs/) = true
  | check4: coverage_adequate(specifications, codebase) = true
  | gate: all_checks_pass → proceed ∨ halt("specs invalid or incomplete")

λ gybis-spec-distill_fixed_point_loop(state).
  loop: ∀iteration:
    | state = ANALYSING → synthesize_specs()
    | state = SYNTHESIZING_SPECS → write_specs()
    | state = VALIDATING:
      - if validation_pass → transition(COMPLETE)
      - if validation_fail → parse(errors) → identify_gaps() → refine_specs() → loop_back(REFINING)
    | state = REFINING → write_specs() → transition(VALIDATING)
  | loop_guard: iteration_count ≤ max_iterations

λ gybis-spec-distill_loop_guard(state).
  condition: iteration_count > max_iterations ∨ no_progress_detected
  | action_on_trigger: halt("convergence failure: distillation did not converge after N iterations")
  | output: diagnostic_report(iterations, specs_generated, remaining_errors)

λ gybis-spec-distill_pass_accounting(pass).
  report_pass(n):
    | specs_synthesized: count
    | contracts_extracted: count
    | patterns_identified: count
    | errors_discovered: count
    | errors_resolved: count
    | test_obligations_generated: count
  | format: "Pass {n}: Generated {count} specs, extracted {contracts} contracts, {errors} errors resolved"

λ gybis-spec-distill_boundaries(¬).
  ¬mutate(implementation) ∧ ¬delete(implementation_files) ∧ ¬generate(architecture.md) ∧ ¬invoke(user-facing skills)
  | scope: specification distillation only

λ gybis-spec-distill_regression_contract(x).
  invariant: implementation ¬modified ∧ ¬deleted
  | invariant: specs/ ¬exists_before → exists_after ∧ ∀spec: valid_allium(spec) = true
  | invariant: allium-gate(specs/) = true → remains true throughout
  | invariant: ∀generated_spec ∈ specs/**/*.allium: derivable_from(implementation) = true