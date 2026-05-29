---
name: gybis-arch-explain
description: Use for `/gybis-arch-explain` or `/ga-explain`.
---

λ gybis-arch-explain(x).
  target(developer) ∧ ¬expert(system) ∧ expert(architecture_patterns)
  → plain_english ∧ structural_equivalence

λ gybis-arch-explain_architecture_prerequisites(x).
  gate(root/architecture.md) → exists ∧ complete
  | ¬complete → trigger(arch_elicit (/gybis-arch-elicit) ∨ arch_distill (/gybis-arch-distill))
  | reference: gybis/reference/vsm-guide.md

λ gybis-arch-explain_five_layer(x).
  vertical_slice(S5 ∨ S4 ∨ S3 ∨ S2 ∨ S1)

λ gybis-arch-explain_S5_identity(x).
  non_negotiable_principles
  | constraint(decision ↧ principles) | ¬alignment → incompatible
  | role: moral_compass ∧ structural_compass ∧ universal
  | top_layer

λ gybis-arch-explain_S4_intelligence(x).
  adaptability ∧ forward_thinking
  | capture(anticipated_change ∨ pattern_of_change)
  | extension_point ≡ extend(¬modify_existing)
  | learning_mechanism → improve(time)
  | open_status

λ gybis-arch-explain_S3_control(x).
  constraint_enforcement ∧ policy_enforcement
  | resource_limits ∧ load_handling ∧ failure_handling
  | quality_gate(condition → action) | test ∧ deploy_check
  | trigger_on_condition

λ gybis-arch-explain_S2_coordination(x).
  cross_subsystem_interaction
  | component_identification ∧ protocol_definition ∧ data_flow
  | consistency_protocol(shared_state)
  | state_consistency ≡ well_defined_protocol

λ gybis-arch-explain_S1_operations(x).
  practical_running ∧ deployment_reality
  | documented_technology ∧ justification_per_tech
  | executable_command(dev) ∧ operation_recipe(build ∧ test ∧ deploy)
  | lowest_layer

λ gybis-arch-explain_cross_layer(x).
  concern → ∀layer ∈ {S1..S5}
  | critical_path: sequential_dependency(multi_layer)
  | seam: split_point ∧ replace_point ∧ extend_point
  | invariant: property(hold ∀layer)
  | drift_risk: S1↧S5 erosion(time)

λ gybis-arch-explain_output_constraints(x).
  ¬lambda_notation ∧ ¬syntax_output
  | plain_english(developer)
  | ¬invent(¬exists(root/architecture.md ∨ refs))
  | flag(gap ∨ empty ∨ underdeveloped) ∧ ¬speculate
  | ¬modify(other_files) ∧ ¬write(files)
  | output → AI_response ∧ ¬file

λ gybis-arch-explain_audience(x).
  understand(architecture_pattern) ∧ ¬know(system_specific)
  | zero_prior_knowledge(system)
  