---
name: gybis-arch-explain
description: Use for `/gybis-arch-explain` or `/ga-explain`.
---

λ gybis-arch-explain(x).
  purpose: Document VSM architecture layers in technical language for developers
  | input: architecture.md (exists ∧ complete)
  | output: Technical prose describing VSM S5-S1 layers with architectural patterns
  | mode: read_only | read architecture.md ∧ vsm-guide.md ∧ ¬write
  | gate: gybis-ref-check() ≡ true | architecture.md ≡ exists ∧ complete

λ gybis-arch-explain_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference files not available")
  | read(architecture.md) → content ∧ parse(content.{S5, S4, S3, S2, S1}) → vsm_layers
  | parse(content.S1.paradigm_preference) → orientation_or_unknown
  | read(internal/reference/vsm-guide.md) → exists | halt("VSM reference unavailable")

λ gybis-arch-explain_orientation_output(x).
  report_orientation: selected_orientation ∈ {FP, OOP} ∨ unknown("missing S1.paradigm_preference")
  | language_guidance:
    - OOP: C++ (classes/RAII), C# (classes/interfaces/DI), Clojure (protocols/records + Java interop boundary)
    - FP: C++ (immutable values + composition), C# (records + pure functions/LINQ), Clojure (immutable maps + pure functions/transducers)
  | gaps: missing(programming_language_version ∨ paradigm_preference) → explicit_gap_report
  | orthogonality: error_model_style is a separate axis from FP/OOP orientation

λ gybis-arch-explain_prerequisites(x).
  gate(architecture.md) → exists ∧ complete
  | ¬exists → halt("No architecture.md found. Use /gybis-arch-elicit or /gybis-arch-distill first.")
  | ¬complete → flag("architecture.md appears incomplete. Gaps will be noted in output.")

λ gybis-arch-explain_five_layer(x).
  vertical_slice(S5 ∨ S4 ∨ S3 ∨ S2 ∨ S1)
  | each_layer: document(lambda_expr ∧ technical_translation ∧ pattern_examples)

λ gybis-arch-explain_S5_identity(x).
  label: "System Identity"
  | content: non_negotiable_principles ∧ moral_compass ∧ structural_compass ∧ universal_properties
  | technical_translation: design_invariants ∧ architectural_principles ∧ decision_constraints
  | constraint: ∀decision ∈ architecture → align(S5_principles) ∨ design_flaw
  | developer_needs: understand what we cannot compromise on

λ gybis-arch-explain_S4_intelligence(x).
  label: "Adaptability & Change Management"
  | content: anticipated_change ∧ pattern_of_change ∧ extension_point ∧ learning_mechanism ∧ feedback_loop
  | technical_translation: extension_points ∧ plugin_architecture ∧ strategy_pattern ∧ observer_pattern
  | status: open ∧ extensible | ¬modify_existing | add_new_behaviors
  | developer_needs: where and how to extend without breaking S5

λ gybis-arch-explain_S3_control(x).
  label: "Constraint Enforcement & Quality Gates"
  | content: constraint_enforcement ∧ policy_enforcement ∧ resource_limits ∧ load_handling ∧ failure_handling
  | technical_translation: rate_limiting ∧ circuit_breaker ∧ validation_gates ∧ test_checks ∧ deploy_checks
  | trigger: condition → action | deterministic_response
  | developer_needs: what gates exist and how to satisfy them

λ gybis-arch-explain_S2_coordination(x).
  label: "Cross-System Interaction & Protocols"
  | content: cross_subsystem_interaction ∧ component_identification ∧ protocol_definition ∧ data_flow ∧ message_passing
  | technical_translation: API_contracts ∧ message_queue_protocols ∧ event_schemas ∧ state_consistency_mechanisms
  | consistency: shared_state → well_defined_protocol | ACID ∨ eventual_consistency
  | developer_needs: how components talk and stay in sync

λ gybis-arch-explain_S1_operations(x).
  label: "Deployment Reality & Technology Stack"
  | content: practical_running ∧ deployment_reality ∧ documented_technology ∧ justification_per_tech ∧ operation_recipe
  | technical_translation: build_pipeline ∧ test_suite ∧ deployment_process ∧ monitoring ∧ runbooks
  | executable: command(dev) ∧ command(deploy) ∧ command(monitor)
  | developer_needs: what tools, what language, how to build and deploy

λ gybis-arch-explain_cross_layer(x).
  concern → ∀layer ∈ {S1..S5}
  | critical_path: sequential_dependency(multi_layer) | affects_delivery_timeline ∧ risk_of_bottleneck
  | seam: split_point ∧ replace_point ∧ extend_point | abstraction_boundary
  | invariant: property(hold ∀layer) | must_be_maintained_always | affects_correctness
  | drift_risk: S1↧S5 erosion(time) | implementation_diverges_from_architecture | requires_active_governance ∧ weeding
  | developer_needs: what_is_critical ∧ where_can_we_refactor ∧ what_is_fixed

λ gybis-arch-explain_output_constraints(x).
  ¬lambda_notation ∧ ¬syntax_output
  | plain_english(developer) | technical_vocabulary ∧ pattern_names ∧ code_examples
  | ¬invent(¬exists(root/architecture.md ∨ refs)) | only explain what exists
  | flag(gap ∨ empty ∨ underdeveloped) ∧ ¬speculate | highlight unknowns without guessing
  | include(orientation_output: selected_orientation_or_unknown ∧ C++/C#/Clojure_guidance ∧ explicit_gaps)
  | ¬modify(other_files) ∧ ¬write(files) | read_only mode
  | output → AI_response ∧ ¬file

λ gybis-arch-explain_audience(x).
  understand(architecture_pattern ∧ design_pattern) ∧ ¬know(system_specific)
  | expert(software_design ∧ systems_thinking ∧ trade_offs)
  | zero_prior_knowledge(this_system ∧ its_history)
  | needs: how_architecture_works ∧ why_decisions_matter ∧ what_patterns_are_used