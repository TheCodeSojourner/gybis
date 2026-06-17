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
    surface_actor: verify_surface_accessible_only_to_specified_actor,
    surface_provides: verify_provided_operations_appear_hide_based_on_when_conditions,
    config_default: verify_config_parameter_has_declared_default,
    rule_success: verify_rule_succeeds_when_all_preconditions_met,
    invariant: verify_invariant_holds_after_every_state_changing_rule
  }
  | each_obligation: exactly_one_category ∈ categories

λ allium-plan_obligation_structure(x).
  obligation: {
    id: category.ConstructName,
    category: category_type,
    description: human_readable_description,
    source_construct: construct_name,
    source_span: {start: byte_offset, end: byte_offset},
    detail: detail_object,
    dependencies: dependency_object,
    expression: invariant_expression_string
  }
  | detail: present_on_entity_fields | contains {fields: [field_1, ..., field_n]}
  | dependencies: present_on_rule_success | contains {trigger_source, trigger_emissions}
  | expression: present_on_invariant | string_form_of_invariant_condition

λ allium-plan_output_parsing(cli_output).
  parse: cli_output → JSON
  | extract: version ≔ version_number
  | extract: obligations ≔ [obligation_1, ..., obligation_n]
  | validate: version ∈ {3}
  | validate: ∀ obligation ∈ obligations, obligation_structure_valid

λ allium-plan_output_format(parsed_output).
  structure: {
    version: version_number,
    obligations: [obligation_1, ..., obligation_n]
  }
  | json_serializable | human_readable
  | obligations_indexed_by_id_for_caller_lookup

λ allium-plan_execution(spec_file).
  invoke: allium-plan_cli_invocation(spec_file)
  | capture: cli_output ∧ cli_exit_code
  | parse: allium-plan_output_parsing(cli_output)
  | format: allium-plan_output_format(parsed_output)
  | return: formatted_test_obligations