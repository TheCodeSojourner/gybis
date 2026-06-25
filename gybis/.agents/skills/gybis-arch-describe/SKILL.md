---
name: gybis-arch-describe
description: Use for `/gybis-arch-describe` or `/ga-describe`.
---

λ gybis-arch-describe(x).
  purpose: Document VSM architecture layers in plain English for product managers
  | input: architecture.md (exists ∧ complete)
  | output: Plain English prose describing VSM S5-S1 layers
  | mode: read_only | read architecture.md ∧ vsm-guide.md ∧ ¬write
  | gate: gybis-ref-check() ≡ true | architecture.md ≡ exists ∧ complete

λ gybis-arch-describe_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference files not available")
  | read(architecture.md) → content ∧ parse(content.{S5, S4, S3, S2, S1}) → vsm_layers
  | parse(content.S1.paradigm_preference) → orientation_or_unknown
  | read(internal/reference/vsm-guide.md) → exists | halt("VSM reference unavailable")

λ gybis-arch-describe_orientation_output(x).
  report_orientation: selected_orientation ∈ {FP, OOP} ∨ unknown("missing S1.paradigm_preference")
  | language_guidance:
    - OOP: C++ (classes/RAII), C# (classes/interfaces/DI), Clojure (protocols/records + Java interop boundary)
    - FP: C++ (immutable values + composition), C# (records + pure functions/LINQ), Clojure (immutable maps + pure functions/transducers)
  | gaps: missing(programming_language_version ∨ paradigm_preference) → explicit_gap_report
  | orthogonality: error_model_style is a separate axis from FP/OOP orientation

λ gybis-arch-describe_prerequisites(x).
  gate(architecture.md) → exists ∧ complete
  | ¬exists → halt("No architecture.md found. Use /gybis-arch-elicit or /gybis-arch-distill first.")
  | ¬complete → flag("architecture.md appears incomplete. Gaps will be noted in output.")

λ gybis-arch-describe_five_layer(x).
  vertical_slice(S5 ∨ S4 ∨ S3 ∨ S2 ∨ S1)
  | each_layer: document(lambda_expr ∧ plain_english_translation)

λ gybis-arch-describe_S5_identity(x).
  label: "System Identity & Values"
  | content: non_negotiable_principles ∧ moral_compass ∧ structural_compass
  | business_translation: vision ∧ core_values ∧ non_negotiables
  | constraint: ∀decision ∈ architecture → align(S5_principles) ∨ incompatible

λ gybis-arch-describe_S4_intelligence(x).
  label: "Adaptability & Growth"
  | content: anticipated_change ∧ pattern_of_change ∧ extension_point ∧ learning_mechanism
  | business_translation: innovation_opportunities ∧ growth_pathways ∧ evolution_readiness
  | status: open ∧ extensible | ¬frozen

λ gybis-arch-describe_S3_control(x).
  label: "Governance & Compliance"
  | content: constraint_enforcement ∧ policy_enforcement ∧ resource_limits ∧ quality_gate
  | business_translation: rules_of_engagement ∧ guardrails ∧ accountability
  | trigger: condition → action

λ gybis-arch-describe_S2_coordination(x).
  label: "Team Collaboration & Handoffs"
  | content: cross_subsystem_interaction ∧ component_identification ∧ protocol_definition ∧ data_flow
  | business_translation: how_parts_work_together ∧ communication_patterns ∧ integration_points
  | consistency: shared_state → well_defined_protocol

λ gybis-arch-describe_S1_operations(x).
  label: "Day-to-Day Work & Tools"
  | content: practical_running ∧ deployment_reality ∧ documented_technology ∧ operation_recipe
  | business_translation: what_we_use ∧ how_we_build ∧ how_we_deploy ∧ how_we_maintain
  | foundation: build ∧ test ∧ deploy

λ gybis-arch-describe_cross_layer(x).
  concern → ∀layer ∈ {S1..S5}
  | critical_path: sequential_dependency(multi_layer) | affects_delivery_timeline
  | seam: split_point ∧ replace_point ∧ extend_point | risk_of_tight_coupling
  | invariant: property(hold ∀layer) | cannot_be_violated
  | drift_risk: S1↧S5 erosion(time) | require_active_maintenance ∧ governance
  | business_translation: what_affects_everything ∧ where_can_we_flex ∧ what_cannot_break

λ gybis-arch-describe_output_constraints(x).
  ¬lambda_notation ∧ ¬syntax_output
  | plain_english(product_manager) | business_vocabulary ∧ concrete_examples
  | ¬invent(¬exists(root/architecture.md ∨ refs)) | only describe what exists
  | flag(gap ∨ empty ∨ underdeveloped) ∧ ¬speculate | highlight unknowns without guessing
  | include(orientation_output: selected_orientation_or_unknown ∧ C++/C#/Clojure_guidance ∧ explicit_gaps)
  | ¬modify(other_files) ∧ ¬write(files) | read_only mode
  | output → AI_response ∧ ¬file

λ gybis-arch-describe_audience(x).
  understand(business_context) ∧ ¬know(system_specific)
  | expert(vision ∧ goals ∧ business ∧ constraints)
  | zero_prior_knowledge(technical_architecture ∧ vsm)
  | needs: what_system_is ∧ why_it_matters ∧ what_drives_decisions