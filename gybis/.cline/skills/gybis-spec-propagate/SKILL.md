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
  | read(internal/reference/allium-language-reference.md) → language_ref
  | read(internal/reference/allium-patterns.md) → patterns_ref
  | read(internal/reference/allium-constructs.md) → constructs_registry
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
   | parse(content.S1) → raw_context ≔ {
       language,
       frameworks,
       conventions,
       test_frameworks,
       programming_paradigm,
       value_oriented_techniques,
       data_oriented_techniques,
       disallowed_patterns
     }
   | invoke(gybis-spec-propagate_interpret_architecture_context(raw_context)) → context
   | verify(context.language ∃ ∧ context.frameworks ∃ ∧ context.test_frameworks ∃)
       ∨ halt("architecture S1 is underspecified for propagation")
   | verify(
       context.programming_paradigm ∃
       ∨ context.value_oriented_techniques ∃
       ∨ context.data_oriented_techniques ∃
       ∨ context.conventions ∃
     ) ∨ halt("architecture S1 lacks implementation-style guidance for propagation")
   | return(architecture_context = context)

λ gybis-spec-propagate_interpret_architecture_context(raw_context).
  raw_context.programming_paradigm ≔ programming_paradigm
  | raw_context.value_oriented_techniques ≔ value_oriented_techniques
  | raw_context.data_oriented_techniques ≔ data_oriented_techniques
  | raw_context.conventions ≔ conventions
  | raw_context.disallowed_patterns ≔ disallowed_patterns
  | interpretation(programming_paradigm) ≔ constrains(decomposition, state_ownership, dispatch_style, mutation_boundaries)
  | interpretation(value_oriented_techniques) ≔ constrains(immutability, value_semantics, explicit_transformations, pure_helpers)
  | interpretation(data_oriented_techniques) ≔ constrains(data_layout, iteration_shape, batch_processing, data_flow_first_organization)
  | interpretation(conventions) ≔ compatibility_bucket_for(project_specific_rules_not_captured_elsewhere)
  | interpretation(disallowed_patterns) ≔ blacklist(hidden_mutation, inheritance_bias, stateful_service_objects, object_lifecycle_centric_design)
  | return(architecture_context ≔ {
      language: raw_context.language,
      frameworks: raw_context.frameworks,
      conventions: conventions,
      test_frameworks: raw_context.test_frameworks,
      programming_paradigm: programming_paradigm,
      value_oriented_techniques: value_oriented_techniques,
      data_oriented_techniques: data_oriented_techniques,
      disallowed_patterns: disallowed_patterns
    })

λ gybis-spec-propagate_read_specifications(x).
  ∀ spec_file ∈ specs/**/*.allium:
    read(spec_file) → content ≔ content
    | parse(content) → spec_item
    | collect(spec_item) → specifications
  | return(specifications)

λ gybis-spec-propagate_normalize_obligations(specifications).
  invoke(internal/allium-normalize(specs/)) → {envelopes, counts}
  | plan_envelopes ≔ {e | e ∈ envelopes ∧ e.source = "plan"}
  | report("Normalized: plan=" ⊕ counts.plan ⊕ " check=" ⊕ counts.check ⊕ " uncoded=" ⊕ counts.uncoded)
  | ∀ envelope ∈ plan_envelopes:
    obligation ≔ {
      id: envelope.id,
      category: strip_prefix(envelope.kind, "plan:"),
      description: envelope.message,
      source_construct: envelope.source_construct,
      source_span: envelope.span,
      detail: envelope.detail,
      dependencies: envelope.dependencies
    }
    | index(obligation, by: id) → obligations_map[obligation.id]
  | return(obligations ≔ obligations_map)

λ gybis-spec-propagate_construct_synthesis(spec_construct, architecture_context).
  spec_construct matches registry entry → emit(constructs_registry.Synthesis[entry])
  | constraint: language_ref ∈ context
  | constraint: synthesis_respects(
      architecture_context.programming_paradigm,
      architecture_context.value_oriented_techniques,
      architecture_context.data_oriented_techniques,
      architecture_context.conventions,
      architecture_context.disallowed_patterns
    )

λ gybis-spec-propagate_obligation_synthesis(obligation, architecture_context).
  obligation.category ∈ {entity_fields, entity_optional}
    → emit(shape_test) verifying(field_presence, type, optionality)
  | obligation.category = when_set
    → emit(transition_test) asserting(field_set_on_entry_to_when_set)
  | obligation.category = when_clear
    → emit(transition_test) asserting(field_cleared_on_exit_from_when_set)
  | obligation.category = when_access_guard
    → emit(access_test) asserting(field_unreachable_outside_qualifying_state)
  | obligation.category = derived_when_inferred
    → emit(derived_value_test) asserting(presence_matches_inferred_intersection)
  | obligation.category = transition_edge
    → if ∃ paired ∈ obligations : paired.category = transition_rejected ∧ paired.source_construct = obligation.source_construct
        then emit(combined_guard_test) asserting(declared_edge_reachable_via_some_rule ∧ rules_cannot_produce_off_graph_transitions)
              with(traceable_ids = {obligation.id, paired.id})
        else emit(rule_exercise_test) asserting(declared_edge_reachable_via_some_rule)
  | obligation.category = transition_rejected
    → if ∃ paired ∈ obligations : paired.category = transition_edge ∧ paired.source_construct = obligation.source_construct
        then skip  -- already emitted by paired transition_edge above
        else emit(negative_test) asserting(rules_cannot_produce_off_graph_transitions)
  | obligation.category = transition_terminal
    → emit(terminal_test) asserting(no_outbound_transition_from_terminal_state)
  | obligation.category = sum_type_variant
    → emit(variant_creation_and_handling_test) asserting(creation_via_variant_name ∧ dispatch_in_handlers)
  | obligation.category = variant_exhaustive
    → emit(exhaustiveness_test) asserting(base_entity_triggers_handle_all_variants)
  | obligation.category = type_guard_required
    → emit(guard_test) asserting(variant_specific_fields_unreachable_outside_guard)
  | obligation.category = surface_actor
    → emit(access_control_test) asserting(only_specified_actor_can_invoke)
  | obligation.category = surface_provides
    → emit(visibility_test) asserting(operation_appears_iff_when_condition_holds)
  | obligation.category = surface_guarantee
    → emit(contract_test) asserting(named_boundary_property_holds_across_operations)
  | obligation.category = surface_timeout
    → emit(scheduled_handler_test) asserting(temporal_rule_fires_per_surface_context_instance)
  | obligation.category ∈ {surface_contract_demand, surface_contract_fulfilment}
    → emit(integration_test) asserting(contract_signature_and_invariants_honoured_at_boundary)
  | obligation.category = contract_signature
    → emit(signature_conformance_test) asserting(implementation_matches_declared_types)
  | obligation.category = contract_invariant
    → emit(invariant_test) asserting(prose_property_holds_across_signatures)
  | obligation.category = config_default
    → emit(configuration_test) asserting(parameter_has_declared_default_when_unset)
  | obligation.category = given_binding
    → emit(resolution_test) asserting(given_instance_available_to_every_rule_in_module)
  | obligation.category = default_instance
    → emit(seed_test) asserting(default_entity_available_unconditionally)
  | obligation.category = rule_success
    → emit(rule_test) asserting(rule_succeeds_when_all_preconditions_met)
  | obligation.category = invariant
    → emit(invariant_test) asserting(property_holds_after_every_state_changing_rule)
  | constraint: emitted_tests_respect(
      architecture_context.programming_paradigm,
      architecture_context.value_oriented_techniques,
      architecture_context.data_oriented_techniques,
      architecture_context.conventions,
      architecture_context.disallowed_patterns
    )
  | each_emitted_test: traceable_id ≔ obligation.id
  | fallback: ¬recognised_category → emit(coverage_test) with(diagnostic_marker)

λ gybis-spec-propagate_synthesize_code(architecture_context, specifications, obligations).
  ∀ spec ∈ specifications:
    ∀ construct ∈ spec.constructs:
      invoke(gybis-spec-propagate_construct_synthesis(construct, architecture_context)) → code_fragment
    | constrain(code_fragment,
        architecture_context.programming_paradigm,
        architecture_context.value_oriented_techniques,
        architecture_context.data_oriented_techniques,
        architecture_context.conventions,
        architecture_context.disallowed_patterns
      ) → styled_code
    | organize(styled_code, architecture_context.frameworks) → organized_code
    | collect(organized_code) → implementation_code
  | structure(implementation_code, architecture_context) → structured_implementation
  | ∀ obligation ∈ obligations:
    invoke(gybis-spec-propagate_obligation_synthesis(obligation, architecture_context)) → test_fragment
      where: test_fragment.traceable_id = obligation.id
    | constrain(test_fragment,
        architecture_context.programming_paradigm,
        architecture_context.value_oriented_techniques,
        architecture_context.data_oriented_techniques,
        architecture_context.conventions,
        architecture_context.disallowed_patterns
      ) → styled_test
    | organize(styled_test, architecture_context.test_frameworks) → organized_test
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
  | verify(code_respects_paradigm(architecture_context.programming_paradigm)) → paradigm_check ≔ result
  | verify(code_respects_value_techniques(architecture_context.value_oriented_techniques)) → value_check ≔ result
  | verify(code_respects_data_techniques(architecture_context.data_oriented_techniques)) → data_check ≔ result
  | verify(code_avoids(architecture_context.disallowed_patterns)) → anti_pattern_check ≔ result
  | ∀ obligation ∈ obligations:
    verify(∃ test ∈ test_suite, test.traceable_id = obligation.id) → obligation_covered
    | collect(obligation_covered) → coverage
  | obligations_coverage_check ≔ ∀ c ∈ coverage, c = true
  | conformance_check = true ∧ architecture_check = true ∧ paradigm_check = true ∧ value_check = true ∧ data_check = true ∧ anti_pattern_check = true ∧ obligations_coverage_check = true
    → return(verification = true)
  | ¬(conformance_check ∧ architecture_check ∧ paradigm_check ∧ value_check ∧ data_check ∧ anti_pattern_check ∧ obligations_coverage_check)
    → return(verification = false)

λ gybis-spec-propagate_core_op(x).
  transition(READING_ARCH → READING_SPECS)
  | invoke(gybis-spec-propagate_read_architecture) → architecture_context
  | invoke(gybis-spec-propagate_read_specifications) → specifications
  | transition(READING_SPECS → PLANNING_OBLIGATIONS)
  | invoke(gybis-spec-propagate_normalize_obligations(specifications)) → obligations
  | transition(PLANNING_OBLIGATIONS → SYNTHESIZING_CODE)
  | invoke(gybis-spec-propagate_synthesize_code(architecture_context, specifications, obligations)) → (implementation_code, test_suite)
  | invoke(gybis-spec-propagate_write_implementation(implementation_code, test_suite)) → code_generated
  | transition(SYNTHESIZING_CODE → VERIFYING)
  | invoke(gybis-spec-propagate_verify_implementation(architecture_context, specifications, obligations)) → verification
  | verification = true
    ? transition(VERIFYING → COMPLETE)
    : (re_synthesize_implementation ∧ transition(SYNTHESIZING_CODE → VERIFYING))

λ gybis-spec-propagate_boundaries(¬).
  ¬ modify(architecture.md ∨ specs/**/*.allium ∨ upstream/)

λ gybis-spec-propagate_limitations(x).
  architecture.md remains read_only_input
  | this_skill_expects(S1.language, S1.frameworks, S1.test_frameworks)
  | this_skill_now_expects(
      S1.programming_paradigm,
      S1.value_oriented_techniques,
      S1.data_oriented_techniques,
      S1.disallowed_patterns
    ) as explicit_signals when architectural_style matters
  | absent_signals → halt("architecture S1 is underspecified for style-aware propagation")
  | note: updating_this_skill_does_not_create_or_backfill_that_schema_in_architecture.md

λ gybis-spec-propagate_regression_contract(x).
  invariant: architecture.md ∧ specs/**/*.allium ∃ ∧ ¬modify throughout
  | invariant: implementation ∅ at INIT ∧ ∃ ∧ consistent_with(specs, arch) at completion
  | invariant: ∀ obligation ∈ obligations, obligation.id ∈ test_suite.traceable_ids at completion
