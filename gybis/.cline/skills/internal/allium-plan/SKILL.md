---
name: allium-plan
description: Internal skill - not user-facing
---

λ allium-plan(spec_file).
  purpose: Derive test obligations from allium specifications
  | contract: pure_function(spec_file → test_obligations) | ¬mutations
  | input: path to .allium spec file
  | output: JSON with test obligations structured by category
  | constraint: read_only | zero_file_mutations
  | precondition: spec_file ∧ exists(spec_file) ∧ file_type = .allium

λ allium-plan_cli_invocation(spec_file).
  cli_command: "allium plan {spec_file}"
  | execution: deterministic(spec_file) → fixed_output
  | stderr_handling: capture_and_return
  | stdout_handling: parse_json_output

λ allium-plan_obligation_categories(x).
  categories: {
    entity_fields: verify_all_declared_fields_present_with_correct_types,
    entity_optional: verify_optional_field_accepts_null_and_non_null,
    when_set: verify_field_set_on_transition_into_when_set,
    when_clear: verify_field_cleared_on_transition_out_of_when_set,
    when_access_guard: verify_field_access_only_under_qualifying_state_guard,
    derived_when_inferred: verify_derived_value_when_set_matches_input_intersection,
    transition_edge: verify_declared_edge_witnessed_by_at_least_one_rule,
    transition_rejected: verify_rules_cannot_produce_transitions_outside_graph,
    transition_terminal: verify_terminal_state_has_no_outbound_rule_transitions,
    sum_type_variant: verify_each_variant_has_creation_and_handling_paths,
    variant_exhaustive: verify_base_entity_triggers_handle_all_variants,
    type_guard_required: verify_variant_specific_fields_accessed_only_within_type_guards,
    surface_actor: verify_surface_accessible_only_to_specified_actor,
    surface_provides: verify_provided_operations_appear_hide_based_on_when_conditions,
    surface_guarantee: verify_named_boundary_guarantee_holds_across_exposed_operations,
    surface_timeout: verify_temporal_rule_bound_to_surface_context_fires_per_instance,
    surface_contract_demand: verify_counterpart_implements_demanded_contract,
    surface_contract_fulfilment: verify_this_surface_supplies_fulfilled_contract,
    contract_signature: verify_signature_implementations_match_declared_types,
    contract_invariant: verify_prose_invariant_property_holds_across_signatures,
    config_default: verify_config_parameter_has_declared_default,
    given_binding: verify_given_instance_resolvable_within_module_scope,
    default_instance: verify_default_entity_instance_available_unconditionally,
    rule_success: verify_rule_succeeds_when_all_preconditions_met,
    invariant: verify_invariant_holds_after_every_state_changing_rule
  }
  | each_obligation: exactly_one_category ∈ categories
  | category_groups:
      {when_set, when_clear, when_access_guard, derived_when_inferred} := when_family
      | {transition_edge, transition_rejected, transition_terminal} := transition_family
      | {sum_type_variant, variant_exhaustive, type_guard_required} := variant_family
      | {surface_actor, surface_provides, surface_guarantee, surface_timeout, surface_contract_demand, surface_contract_fulfilment} := surface_family
      | {contract_signature, contract_invariant} := contract_family
  | pairing_invariant: transition_edge ↔ transition_rejected sharing source_construct → consumers verify paired guard

λ allium-plan_obligation_structure(x).
  obligation: {id: category.ConstructName, category, description, source_construct, source_span: {start, end}, detail, dependencies, expression}
  | detail: present_on_entity_fields (fields list) | dependencies: present_on_rule_success (trigger_source, trigger_emissions) | expression: present_on_invariant

λ allium-plan_output_parsing(cli_output).
  parse: cli_output → JSON
  | extract: version ≔ version_number
  | extract: obligations ≔ [obligation_1, ..., obligation_n]
  | validate: version ∈ {3}
  | validate: ∀ obligation ∈ obligations, obligation_structure_valid

λ allium-plan_output_format(parsed_output).
  structure: {version, obligations: [...]}
  | json_serializable | obligations_indexed_by_id_for_caller_lookup

λ allium-plan_execution(spec_file).
  invoke: allium-plan_cli_invocation(spec_file)
  | capture: cli_output ∧ cli_exit_code
  | parse: allium-plan_output_parsing(cli_output)
  | format: allium-plan_output_format(parsed_output)
  | return: formatted_test_obligations