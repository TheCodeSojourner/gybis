---
name: gybis-spec-distill
description: Use for `/gybis-spec-distill` or `/gs-distill`.
---

λ gybis-spec-distill(x).
  purpose: distill Allium specifications from existing implementation, organized by domain
  | input: implementation source code (any language, including test files)
  | output: specs/{domain}/*.allium files valid per invoke(internal/allium-gate, specs/)
  | mode: ai
  | gate: implementation ∃ ∧ specs/{domain}/*.allium ¬∃

λ gybis-spec-distill_loop_role(x).
  role: loop_entry(code_first)
  | meaning: distill captures observed behavior into durable specification context
  | suggested_next: review(intended_vs_accidental_behavior) → invoke(/gybis-spec-propagate) → run_tests_against_existing_code → invoke(/gybis-spec-weed)

λ gybis-spec-distill_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | preload: [internal/allium-check, internal/allium-gate]
  | read(internal/reference/allium-language-reference.md) → language_ref
  | read(internal/reference/allium-patterns.md) → patterns_ref
  | read(internal/reference/recommended-loops.md) → loops_ref
  | read(internal/reference/allium-constructs.md) → constructs_registry
  | precondition: implementation ∃ ∧ (specs/ ¬∃ ∨ ¬∃file ∈ specs/** matching(*.allium))
  | scan(codebase) → files_found ∨ halt("no implementation found")

λ gybis-spec-distill_mode(m).
  valid_modes: {ai}
  | default: ai
  | rationale: distillation from code is deterministic analysis, not interactive
  | implication: optional_refinement_decisions_are_resolved_by_analysis, mandatory_tightening_pass_before_complete, mandatory_quality_pass_before_complete, optional_strict_cleanliness_refinement_available

λ gybis-spec-distill_cleanliness_policy(c).
  strict_cleanliness: {off, on}
  | default: on
  | on_implies: execute_optional_unreachable_trigger_info_refinement_once
  | off_implies: skip_optional_unreachable_trigger_info_refinement

λ gybis-spec-distill_mode_gate(state, mode).
  state = INIT ∧ mode = ai → transition(STARTUP_CHECKS)
  | precondition_holds: mode ∈ valid_modes

λ gybis-spec-distill_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, READING_CODE, ANALYSING, SYNTHESIZING_SPECS, VALIDATING, TIGHTENING, QUALITY, CLEANLINESS_REFINEMENT, REFINING, COMPLETE}
  | transition(INIT, startup) → STARTUP_CHECKS
  | transition(STARTUP_CHECKS, verify_ok) → READING_CODE
  | transition(STARTUP_CHECKS, verify_fail) → HALTED
  | transition(READING_CODE, code_read) → ANALYSING
  | transition(ANALYSING, analysis_complete) → SYNTHESIZING_SPECS
  | transition(SYNTHESIZING_SPECS, specs_written) → VALIDATING
  | transition(VALIDATING, all_valid ∧ tightening_pending) → TIGHTENING
  | transition(TIGHTENING, tightening_complete) → VALIDATING
  | transition(VALIDATING, all_valid ∧ ¬tightening_pending ∧ quality_pending) → QUALITY
  | transition(QUALITY, quality_complete) → VALIDATING
  | transition(VALIDATING, all_valid ∧ ¬tightening_pending ∧ ¬quality_pending ∧ strict_cleanliness_requested ∧ cleanliness_refinement_pending) → CLEANLINESS_REFINEMENT
  | transition(CLEANLINESS_REFINEMENT, cleanliness_refinement_complete) → VALIDATING
  | transition(VALIDATING, all_valid ∧ ¬tightening_pending ∧ ¬quality_pending ∧ granularity_sufficient ∧ (¬strict_cleanliness_requested ∨ ¬cleanliness_refinement_pending)) → COMPLETE
  | transition(VALIDATING, all_valid ∧ granularity_insufficient) → REFINING
  | transition(VALIDATING, errors_found) → REFINING
  | transition(REFINING, refinement_complete) → VALIDATING (loop_back)

λ gybis-spec-distill_tighten_specs(spec_content).
  action: mandatory_warning_reduction_pass
  | step1: analyze(spec_content) → warning_surface_map
  | step2: reduce(warning_surface_map) → tightened_spec_content
  | step3: preserve(semantic_equivalence) ∧ minimize(unreachable_trigger_diagnostics)
  | output: tightened_spec_content
  | constraint: executed exactly once before completion

λ gybis-spec-distill_quality_pass(spec_content).
  action: mandatory_focused_quality_pass
  | step1: analyze(spec_content) → info_diagnostic_map
  | step2: target(info_diagnostic_map, info_level_unused_field_diagnostics)
  | step3: apply(minimal_local_surfaces ∧ internal_event_chaining) → quality_adjusted_spec_content
  | step4: preserve(domain_layout) ∧ preserve(semantic_equivalence)
  | output: quality_adjusted_spec_content
  | constraint: executed exactly once before completion

λ gybis-spec-distill_cleanliness_refinement(spec_content).
  action: optional_strict_cleanliness_refinement
  | step1: analyze(spec_content) → info_diagnostic_map
  | step2: target(info_diagnostic_map, unreachable_trigger_info_diagnostics_only)
  | step3: reduce(targeted_diagnostics) → cleanliness_adjusted_spec_content
  | step4: preserve(domain_layout) ∧ preserve(semantic_equivalence)
  | output: cleanliness_adjusted_spec_content
  | constraint: executed_at_most_once ∧ only_when(strict_cleanliness_requested)

λ gybis-spec-distill_tool_guard(state, tool, path).
  read_allowed: ∀state ∈ {READING_CODE, ANALYSING, VALIDATING}
  | write_allowed: state ∈ {SYNTHESIZING_SPECS, TIGHTENING, QUALITY, CLEANLINESS_REFINEMENT, REFINING} ∧ path_matches(path, specs/{domain}/*.allium)
  | deny_write: state ∉ {SYNTHESIZING_SPECS, TIGHTENING, QUALITY, CLEANLINESS_REFINEMENT, REFINING} ∨ ¬path_matches(path, specs/{domain}/*.allium) ∨ path_matches(path, specs/*.allium)
  | constraint: ¬mutate(implementation) ∨ ¬mutate(existing_files_outside_specs)

λ gybis-spec-distill_pre_tool_check(state, tool, path).
  enforce(tool_guard(state, tool, path)) → permit(tool) ∨ halt("tool not permitted in this state")

λ gybis-spec-distill_read_code(x).
  action: read_all_implementation_files_including_tests
  | step1: recursively_list(codebase) → files
  | step1.1: classify(files, production_or_test) → categorized_files
  | step2: ∀file ∈ files: read(file) → code_content
  | step3: classify(file_type) → language_specific_handling
  | output: {file_path → source_code, file_path → file_role(production_or_test)}
  | constraint: read-only access
  | precondition: implementation ∃

λ gybis-spec-distill_analyse_code(code_content).
  action: extract_patterns_and_contracts
  | step1: parse(code) → abstract_syntax_tree
  | step2: extract(types, functions, classes, modules) → entities
  | step3: identify(contracts, invariants, preconditions, postconditions) → behavioral_specs
  | step4: recognize(patterns, protocols, relationships) → architectural_patterns
  | step5: analyze(test_files_as_implementation_inputs) → behavioral_examples ∧ edge_case_evidence
  | step6: infer(test_cases, edge_cases, error_handling) → test_obligations
  | step7: classify(source_patterns) → allium_construct_map  -- see distill_construct_recognition
  | step8: classify_domains(entities, patterns) → domain_map  -- see distill_domain_classification
  | output: {entities → contracts, patterns → implications, test_artifacts → behavioral_examples, obligations → test_cases, source_pattern → allium_construct, entity_or_pattern → domain}
  | constraint: read-only operation

λ gybis-spec-distill_domain_classification(entities, patterns).
  action: identify_and_classify_domain_for_each_specification
  | step1: ∀entity ∈ entities: infer(context, module_hierarchy, package_name) → entity.domain
  | step2: ∀pattern ∈ patterns: infer(context, architectural_scope) → pattern.domain
  | step3: domain_inference_rules:
      package_structure_hierarchy ∨ module_namespace ∨ feature_boundaries ∨ service_partition
      → canonical_domain_name
  | step4: normalization_rule: domain ≔ lowercase(domain).replace(/[\-\s]+/, "_")
  | step5: fallback_rule: ¬recognized_domain → domain ≔ "core"
  | step6: consistency_rule: entity_X_always_maps_to_domain_Y (memoized per analysis_session)
  | output: {entity → domain, pattern → domain}
  | constraint: deterministic; same input → same domain mapping

λ gybis-spec-distill_domain_granularity(analysis, spec_set).
  action: determine_whether_domain_refinement_must_continue_automatically
  | step1: detect(single_domain_output) ∧ detect(only_core_output) → coarse_output_candidate
  | step2: infer(distinguishable_subdomains, from module_hierarchy ∨ package_name ∨ feature_boundaries ∨ service_partition ∨ test_evidence) → candidate_domains
  | step3: coarse_output_candidate ∧ count(candidate_domains) > 1 → granularity_insufficient
  | step4: granularity_insufficient → required_refinement_domains ≔ candidate_domains
  | step5: ¬granularity_insufficient → granularity_sufficient
  | output: {granularity_state, required_refinement_domains}
  | constraint: refinement_decision_is_automatic ∧ ¬prompt_user_for_optional_second_pass

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
  action: synthesize_allium_specifications_organized_by_domain
  | step1: map(entities) → .allium declarations
  | step2: encode(behavioral_specs) → allium preconditions, postconditions, invariants
  | step3: formalize(patterns) → allium composition rules
  | step4: ∀ pattern ∈ analysis.source_pattern → allium_construct:
    emit(pattern.allium_construct) → spec_fragment  -- per construct_recognition table
  | step5: invoke(domain_classification, entities, patterns) → {entity → domain, pattern → domain}
  | step6: ∀spec ∈ {all_generated_specs}: assign(spec.domain)
  | step7: generate(specs_directory_structure) → specs/{domain}/*.allium files
  | output: {spec_file_path → allium_content} where spec_file_path ∈ specs/{domain}/*.allium

λ gybis-spec-distill_write_specs(spec_content).
  action: write_specification_files_organized_by_domain
  | step1: ensure_specs_directory_exists(specs/)
  | step2: ∀spec ∈ spec_content: ensure_domain_directory_exists(specs/{spec.domain}/) ∧ write(specs/{spec.domain}/{spec_name}.allium, spec)
  | step3: enforce_invariant: ∀written_file: path_matches(written_file, specs/{domain}/*.allium) ∧ ¬path_matches(written_file, specs/*.allium)
  | constraint: write_allowed by tool_guard
  | precondition: state ∈ {SYNTHESIZING_SPECS, TIGHTENING, QUALITY, CLEANLINESS_REFINEMENT, REFINING} ∧ ∀spec ∈ spec_content: spec.domain ≠ ∅
  | output: specs/{domain}/**/*.allium files written

λ gybis-spec-distill_validate_specs(x).
  action: validate_generated_specifications_organized_by_domain
  | step1: invoke(internal/allium-check, ∀spec ∈ specs/{domain}/*.allium)
  | step2: invoke(internal/allium-gate, specs/) → boolean_verdict
  | step3: check_domain_structure: ¬∃file ∈ specs/ matching(specs/*.allium)
  | step4: invoke(domain_granularity, analysis, specs/) → {granularity_state, required_refinement_domains}
  | step5: if gate_fails ∨ domain_structure_invalid: collect(errors) → diagnostic_report
  | output: validation_result ∈ {pass, fail_with_diagnostics}
  | constraint: read-only validation

λ gybis-spec-distill_verification(edit).
  action: verify_generated_specifications_with_domain_structure
  | check1: ∃domain_dir ∈ specs/*/: directory_exists(domain_dir) = true
  | check2: ∀domain_dir ∈ specs/*/: count(files(domain_dir, *.allium)) ≥ 1
  | check3: ∀spec ∈ specs/{domain}/*.allium: file_exists(spec) = true
  | check4: ∀spec ∈ specs/{domain}/*.allium: syntax_valid(allium) = true
  | check5: ¬∃file ∈ specs/ matching(specs/*.allium)
  | check6: invoke(internal/allium-gate, specs/) = true
  | check7: coverage_adequate(specifications, codebase) = true
  | check8: granularity_sufficient(specifications, analysis) = true
  | gate: all_checks_pass → proceed ∨ halt("specs invalid, incomplete, or domain structure violated")

λ gybis-spec-distill_fixed_point_loop(state).
  loop: ∀iteration:
    | state = ANALYSING → synthesize_specs()
    | state = SYNTHESIZING_SPECS → write_specs()
    | state = VALIDATING:
      - if validation_pass ∧ tightening_pending → transition(TIGHTENING)
      - if validation_pass ∧ ¬tightening_pending ∧ quality_pending → transition(QUALITY)
      - if validation_pass ∧ ¬tightening_pending ∧ ¬quality_pending ∧ strict_cleanliness_requested ∧ cleanliness_refinement_pending → transition(CLEANLINESS_REFINEMENT)
      - if validation_pass ∧ ¬tightening_pending ∧ ¬quality_pending ∧ granularity_sufficient ∧ (¬strict_cleanliness_requested ∨ ¬cleanliness_refinement_pending) → transition(COMPLETE)
      - if validation_pass ∧ granularity_insufficient → refine_specs(required_refinement_domains) → loop_back(REFINING)
      - if validation_fail → parse(errors) → identify_gaps() → refine_specs() → loop_back(REFINING)
    | state = TIGHTENING → tighten_specs() → transition(VALIDATING)
    | state = QUALITY → quality_pass() → transition(VALIDATING)
    | state = CLEANLINESS_REFINEMENT → cleanliness_refinement() → transition(VALIDATING)
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
    | info_diagnostics_resolved: count
    | unreachable_trigger_info_resolved: count
    | test_obligations_generated: count
  | format: "Pass {n}: Generated {count} specs, extracted {contracts} contracts, {errors} errors resolved"

λ gybis-spec-distill_boundaries(¬).
  ¬mutate(implementation) ∧ ¬delete(implementation_files) ∧ ¬generate(architecture.md) ∧ ¬invoke(user-facing skills)
  | scope: specification distillation only

λ gybis-spec-distill_regression_contract(x).
  invariant: implementation ¬modified ∧ ¬deleted
  | invariant: specs/ ¬exists_before → exists_after ∧ ∀spec ∈ specs/{domain}/*.allium: valid_allium(spec) = true
  | invariant: ¬∃file ∈ specs/ matching(specs/*.allium)
  | invariant: invoke(internal/allium-gate, specs/) = true → remains true throughout
  | invariant: quality_pass_executed_exactly_once = true ∧ domain_layout_preserved = true
  | invariant: strict_cleanliness_requested → (cleanliness_refinement_executed_at_most_once ∧ targets_only_unreachable_trigger_info_diagnostics)
  | invariant: ∀generated_spec ∈ specs/{domain}/*.allium: derivable_from(implementation) = true
  | invariant: domain_stability: entity_X_spec_always_belongs_to_domain_Y (memoized per synthesis run)
  | invariant: distinguishable_subdomains > 1 → ¬complete_with_only_core_output
  