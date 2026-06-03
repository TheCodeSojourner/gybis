---
name: gybis-spec-propagate
description: Use for `/gybis-spec-propagate` or `/gs-propagate`.
---

λ gybis-spec-propagate().
  purpose: specs(root/specs/**/*.allium) → source_and_tests | accept(domain_concern ∨ domain ∨ all_specs)

λ gybis-spec-propagate_io_contract.
  read_scope ⊆ (root/specs/**/*.allium ∪ root/src/**/* ∪ root/**/tests/**/*)
  | write_scope ⊆ (root/src/**/* ∪ root/**/tests/**/*)
  | edit_scope ⊆ (root/src/**/* ∪ root/**/tests/**/*)
  | ¬write(root/specs/** ∨ root/**/*.md ∨ root/**/*.txt)

λ gybis-spec-propagate_input_resolve(x).
  domain_concern → root/specs/{domain}/{concern}.allium
  | domain → root/specs/{domain}/*.allium
  | all_specs → root/specs/*/*.allium

λ gybis-spec-propagate_prerequisites.
  gate(`allium --version` ∧ executes) ∧ gate(`allium --version` ∧ executes ∧ version(≥3)) ∧ gate(input_resolves) → pipeline_start
  | ¬gate → halt | recommend([juxt/allium_tools](https://github.com/juxt/allium-tools))

λ gybis-spec-propagate_S0_init.
  parse_resolve(input) → file-set{f₁, f₂, ..., fₙ}
  | verify(codebase ∨ tests_exist)
  | `allium plan {fᵢ}` → obligations
  | `allium model {fᵢ}` → domain_models

λ gybis-spec-propagate_S1_discover.
  detect(test_framework) | check(pbt_support: fast-check ∨ hypothesis ∨ proptest ∨ quickcheck ∨ streamdata ∨ test.check)
  | discover(test_location ∨ entity_naming ∨ factory_naming ∨ surface_components ∨ test_helpers)
  | detect(time_injection_patterns) | check(cross_module_fixtures)
  | review(existing_tests) → covered_obligations

λ gybis-spec-propagate_S2_map_bridge.
  surface → endpoints ∨ components ∨ handlers
  | rule → service_methods ∨ handlers ∨ fsm_transitions
  | entity → factories ∨ builders ∨ fixtures
  | rule_invoke → api_call ∨ func_call ∨ event_emit
  | postcondition → db_query ∨ return_check ∨ event_verify

λ gybis-spec-propagate_S3_surface_tests.
  for_each(surface):
  | actors → iterate(actors)
  | cases: positive(when-true) ∧ negative(when-false) ∧ failure_scenarios
  | actor_isolation: ¬interfere(other_actors)
  | actor_id: identified_by ∧ within_scope
  | context: ¬specified → absent
  | contracts: demands(pre) ∧ fulfillments(post) ∧ signatures
  | guarantees: @guarantee
  | timeouts: temporal_rules ∧ constraints
  | related: navigation_links

λ gybis-spec-propagate_S4_spec_tests.
  for_each(construct):
  | entity_value: fields ∨ types ∨ optionality ∨ presence ∨ relationships ∨ joins ∨ equality
  | enum: comparability ∨ membership ∨ inline_isolation
  | sum_type: variants ∨ guards ∨ exhaustiveness ∨ construction
  | derived: projection ∨ volatility(now) ∨ collections
  | default: unconditional ∨ fields ∨ cross_refs
  | config: defaults ∨ overrides ∨ mandatory ∨ expressions ∨ qualifiers ∨ chains
  | invariant: post_rule ∨ edge_cases ∨ implications ∨ entity_level
  | rule: success ∨ failure ∨ edge ∨ guards ∨ create/remove/bulk ∨ iterate ∨ let ∨ chains
  | state: valid_transitions ∨ invalid ∨ terminal ∨ transitions_to(explicit) ∨ becomes(implicit) ∨ undeclared_reject
  | temporal: deadline_boundaries ∨ re_fire_prevent ∨ null_under_pressure
  | surface: visibility ∨ availability ∨ actor ∨ scope ∨ context ∨ related
  | contract: signature_conformance ∨ @invariant ∨ obligation_direction
  | cross_module: qualified_entities ∨ external_triggers ∨ type_placeholders
  | cross_rule: duplicate_guards ∨ provision_availability ∨ interactions
  | transition_graph: edge_reachable ∨ undeclared_reject ∨ terminal_no_outbound ∨ non_terminal_exit_paths ∨ enum_edges
  | state_fields: present_per_state ∨ leave_obligations ∨ convergent ∨ guard_access ∨ derived_infer
  | scenario: happy ∨ edge ∨ order_independent
  | data_flow: surface → rule → downstream
  | reachability: initial → terminal
  | deadlock: allium_analyse_findings
  | cross_entity: multi_entity_lifecycle

λ gybis-spec-propagate_S5_test_kind.
  deterministic → assertion
  | invariant_verify → pbt(generate_valid_states → apply_rules → check_invariants)
  | transition_graph → state_machine_walk
  | ¬pbt_available → fallback(assertion)

λ gybis-spec-propagate_S6_action_map.
  for_each(edge in transition_graph):
  | rule_witness(edge) → locate(implementing_code)
  | write_test_action(setup ∨ invoke ∨ verify_target)
  | register(from, to)
  | pbt_walk: random_valid_paths(non_terminal_start)

λ gybis-spec-propagate_S7_temporal_validate.
  detect(clock_injection) → generate(temporal_tests)
  | ¬clock_injection → flag(test_infrastructure_gap)
  | ¬sleep ∨ ¬race_conditions

λ gybis-spec-propagate_S8_cross_module_chains.
  trace(trigger_emission_graph)
  | integration_fixture_exists → reuse
  | wiring_clear ∧ simple → generate(fixture ∧ test)
  | wiring_opaque ∨ complex → skeleton(TODO)

λ gybis-spec-propagate_S9_reuse.
  match(existing_tests, spec_obligations) → covered
  | ¬covered → generate(gap_tests)

λ gybis-spec-propagate_S10_deferred.
  deferred_module ∉ codebase → generate(stub ∨ placeholder_interface)

λ gybis-spec-propagate_S11_verify.
  compilation_check(generated_tests) → syntactically_valid ∧ compiles

λ gybis-spec-propagate_S12_allium_analyse.
  `allium_analyse {root/specs/}` → findings
  | priority(missing_producers ∨ dead_transitions) → test_creation
  | deadlock_detected → test(stuck_state_behavior)

λ gybis-spec-propagate_invariants.
  deterministic_completeness: construct → ≥1_test_obligation
  | pbt → invariant_verification
  | state_machine → transition_graph
  | action_map → required(state_machine)
  | time_injection → required(temporal) | ¬present → flag
  | reuse → ¬replace(working_tests)
  | deferred → stubs
  | cross_module → integration_tests
  | generated → starting_point(human_adjustment)

λ gybis-spec-propagate_process_composition.
  μ = S0 · S1 · S2 · S3 · S4 · S5 · S6 · S7 · S8 · S9 · S10 · S11 · S12
  