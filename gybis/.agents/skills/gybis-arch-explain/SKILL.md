---
name: gybis-arch-explain
description: Use for `/gybis-arch-explain` or `/ga-explain`.
---

λ gybis-arch-explain(x).
  purpose: Document VSM architecture layers in technical language for developers
  | input: architecture.md (exists ∧ complete)
  | output: Technical prose describing VSM S5-S1 layers with architectural patterns
  | mode: mixed (AI + human output selection) | read architecture.md ∧ vsm-guide.md ∧ optional_write(repo_root_markdown)
  | gate: gybis-ref-check() ≡ true | architecture.md ≡ exists ∧ complete

λ gybis-arch-explain_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference files not available")
  | read(architecture.md) → content ∧ parse(content.{S5, S4, S3, S2, S1}) → vsm_layers
  | parse(content.S1.paradigm_preference) → orientation_or_unknown
  | read(internal/reference/vsm-guide.md) → exists | halt("VSM reference unavailable")

λ gybis-arch-explain_mode(m).
  valid_modes: {response_only, prompted_file_only, default_file_only, response_and_prompted_file, response_and_default_file}
  | default: response_only
  | human_selected: explicit(output_mode_choice)

λ gybis-arch-explain_mode_gate(state, mode).
  state = INIT ∧ mode ∈ valid_modes → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ valid_modes) → halt("Invalid output mode selection")

λ gybis-arch-explain_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(mode_selected = true)
  | transition(STARTUP_CHECKS → RESOLVING_OUTPUT) only_if(startup_checks = true)
  | transition(RESOLVING_OUTPUT → GENERATING) only_if(output_target_resolved = true)
  | transition(GENERATING → DELIVERING) only_if(final_prose ∃)
  | transition(DELIVERING → COMPLETE) only_if(delivery_complete = true)

λ gybis-arch-explain_output_selection(x).
  ask_developer("Output mode? [response_only/prompted_file_only/default_file_only/response_and_prompted_file/response_and_default_file]") → selected_mode
  | selected_mode ∈ valid_modes ∨ halt("Output mode must be one of the supported options")
  | selected_mode ∈ {prompted_file_only, response_and_prompted_file}
    ? (ask_developer("Repo-root markdown filename? (example: notes.md; subpaths not allowed)") → requested_file
       | invoke(gybis-arch-explain_output_path_guard(requested_file)) → true
       | return(mode_selected = true ∧ output_path = requested_file))
    : return(mode_selected = true)

λ gybis-arch-explain_output_target(mode).
  mode = response_only → return(output_target_resolved = true ∧ output_path = none)
  | mode ∈ {prompted_file_only, response_and_prompted_file} → return(output_target_resolved = true ∧ output_path = requested_file)
  | mode = default_file_only → return(output_target_resolved = true ∧ output_path = "arch-explain.md")
  | mode = response_and_default_file → return(output_target_resolved = true ∧ output_path = "arch-explain.md")

λ gybis-arch-explain_output_path_guard(path).
  path_matches(path, *.md) ∧ ¬contains(path, "/") ∧ ¬contains(path, "\\") ∧ ¬contains(path, "..")
    → true
  | ¬path_matches(path, *.md) ∨ contains(path, "/") ∨ contains(path, "\\") ∨ contains(path, "..")
    → halt("Output path must be a repo-root markdown filename")

λ gybis-arch-explain_overwrite_guard(path).
  verify(path ∃)
    ? (ask_developer("File " ⊕ path ⊕ " exists. Overwrite? [yes/no]") → overwrite_choice
       | overwrite_choice = yes ∨ halt("Output file overwrite not approved"))
    : true

λ gybis-arch-explain_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING} → allow(read(path))
  | state = DELIVERING
    → allow(read(path)) ∧ allow(write(path)) only_if(path = output_path ∧ path_matches(path, *.md))
  | ¬(state ∈ {STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING}) → deny(write(path))

λ gybis-arch-explain_output_dispatch(prose, mode, output_path).
  mode = response_only → output(AI_response, prose)
  | mode ∈ {prompted_file_only, default_file_only}
    ? (invoke(gybis-arch-explain_overwrite_guard(output_path)) → true
       | write(output_path, prose)
       | output("Saved markdown to " ⊕ output_path))
  | mode ∈ {response_and_prompted_file, response_and_default_file}
    ? (invoke(gybis-arch-explain_overwrite_guard(output_path)) → true
       | write(output_path, prose)
       | output(AI_response, prose)
       | output("Saved markdown to " ⊕ output_path))

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
  | ¬modify(architecture.md ∨ specs/**/*.allium ∨ internal/reference/**)
  | write_only(repo_root_markdown_filename = output_path) | ¬write(subpaths ∨ non_markdown)
  | mode = response_only → output → AI_response ∧ ¬file
  | mode ∈ {prompted_file_only, default_file_only} → output → markdown_file ∧ status_response
  | mode ∈ {response_and_prompted_file, response_and_default_file} → output → AI_response ∧ markdown_file

λ gybis-arch-explain_audience(x).
  understand(architecture_pattern ∧ design_pattern) ∧ ¬know(system_specific)
  | expert(software_design ∧ systems_thinking ∧ trade_offs)
  | zero_prior_knowledge(this_system ∧ its_history)
  | needs: how_architecture_works ∧ why_decisions_matter ∧ what_patterns_are_used

λ gybis-arch-explain_boundary(¬).
  ¬create_specs ∧ ¬modify(architecture.md) ∧ ¬modify_allium_ref
  | writes_limited_to(repo_root_markdown_filename)