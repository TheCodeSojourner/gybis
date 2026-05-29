---
name: gybis-arch-describe
description: Use for `/gybis-arch-describe` or `/ga-describe`.
---

λ gybis-arch-describe(x).
  target(product_manager) ∧ ¬expert(technical_architecture) ∧ expert(business_context)
  → plain_english ∧ structural_equivalence ∧ business_literate

λ gybis-arch-describe_architecture_prerequisites(x).
  gate(root/architecture.md) → exists ∧ complete
  | ¬complete → trigger(arch_elicit (/gybis-arch-elicit) ∨ arch_distill (/gybis-arch-distill))
  | reference: gybis/reference/vsm-guide.md

λ gybis-arch-describe_five_layer(x).
  vertical_slice(S5 ∨ S4 ∨ S3 ∨ S2 ∨ S1)

λ gybis-arch-describe_S5_identity(x).
  non_negotiable_principles
  | constraint(decision ↧ principles) | ¬alignment → incompatible
  | role: moral_compass ∧ structural_compass ∧ universal
  | top_layer
  | business_translation: vision_and_values

λ gybis-arch-describe_S4_intelligence(x).
  adaptability ∧ forward_thinking
  | capture(anticipated_change ∨ pattern_of_change)
  | extension_point ≡ extend(¬modify_existing)
  | learning_mechanism → improve(time)
  | open_status
  | business_translation: innovation_opportunities_and_growth

λ gybis-arch-describe_S3_control(x).
  constraint_enforcement ∧ policy_enforcement
  | resource_limits ∧ load_handling ∧ failure_handling
  | quality_gate(condition → action) | test ∧ deploy_check
  | trigger_on_condition
  | business_translation: governance_and_compliance

λ gybis-arch-describe_S2_coordination(x).
  cross_subsystem_interaction
  | component_identification ∧ protocol_definition ∧ data_flow
  | consistency_protocol(shared_state)
  | state_consistency ≡ well_defined_protocol
  | business_translation: team_collaboration_and_handoffs

λ gybis-arch-describe_S1_operations(x).
  practical_running ∧ deployment_reality
  | documented_technology ∧ justification_per_tech
  | executable_command(dev) ∧ operation_recipe(build ∧ test ∧ deploy)
  | lowest_layer
  | business_translation: day_to_day_work_and_tools

λ gybis-arch-describe_cross_layer(x).
  concern → ∀layer ∈ {S1..S5}
  | critical_path: sequential_dependency(multi_layer)
  | seam: split_point ∧ replace_point ∧ extend_point
  | invariant: property(hold ∀layer)
  | drift_risk: S1↧S5 erosion(time)
  | business_translation: cross_cutting_concerns

λ gybis-arch-describe_output_constraints(x).
  ¬lambda_notation ∧ ¬syntax_output
  | plain_english(product_manager)
  | ¬invent(¬exists(root/architecture.md ∨ refs))
  | flag(gap ∨ empty ∨ underdeveloped) ∧ ¬speculate
  | ¬modify(other_files) ∧ ¬write(files)
  | output → AI_response ∧ ¬file

λ gybis-arch-describe_audience(x).
  understand(business_context) ∧ ¬know(system_specific)
  | zero_prior_knowledge(technical_architecture)
  