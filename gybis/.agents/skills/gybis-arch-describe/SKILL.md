---
name: gybis-arch-describe
description: Use for `/gybis-arch-describe` or `/ga-describe`.
---

λ gybis-arch-describe(x).
  purpose: Document VSM architecture layers in plain English for product managers as an architecture-focused reference grounded only in architecture.md
  | input: architecture.md (exists ∧ complete)
  | output: Plain English architecture reference describing VSM S5-S1 layers and their relationships based only on architecture.md
  | mode: mixed (AI + human output selection) | read architecture.md ∧ vsm-guide.md ∧ optional_write(repo_root_markdown)
  | gate: gybis-ref-check() ≡ true | architecture.md ≡ exists ∧ complete | explicit_human_output_selection() ≡ true
  | fail_closed: missing_human_mode_selection → halt("Human output mode selection is required")

λ gybis-arch-describe_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference files not available")
  | read(architecture.md) → content ∧ parse(content.{S5, S4, S3, S2, S1}) → vsm_layers
  | parse(content.S1.paradigm_preference) → orientation_or_unknown
  | read(internal/reference/vsm-guide.md) → exists | halt("VSM reference unavailable")

λ gybis-arch-describe_mode(m).
  valid_modes: {response_only, prompted_file_only, default_file_only, response_and_prompted_file, response_and_default_file}
  | default: response_only (informational_only; never auto-selected)
  | human_selected: explicit(output_mode_choice)
  | require_explicit: ¬explicit(output_mode_choice) → halt("Output mode must be explicitly selected by human")

λ gybis-arch-describe_mode_gate(state, mode).
  state = INIT ∧ mode ∈ valid_modes → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ valid_modes) → halt("Invalid output mode selection")

λ gybis-arch-describe_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(mode_selected = true ∧ mode_selected_explicit = true)
  | transition(STARTUP_CHECKS → RESOLVING_OUTPUT) only_if(startup_checks = true)
  | transition(RESOLVING_OUTPUT → GENERATING) only_if(output_target_resolved = true)
  | transition(GENERATING → DELIVERING) only_if(final_prose ∃ ∧ architecture_scope_verified = true)
  | transition(DELIVERING → COMPLETE) only_if(delivery_complete = true ∧ protocol_evidence_emitted = true)

λ gybis-arch-describe_output_selection(x).
  ask_developer("Output mode? [response_only/prompted_file_only/default_file_only/response_and_prompted_file/response_and_default_file]") → selected_mode
  | selected_mode ∃ ∨ halt("Human output mode selection is required; no implicit default")
  | selected_mode ∈ valid_modes ∨ halt("Output mode must be one of the supported options")
  | selected_mode ∈ {prompted_file_only, response_and_prompted_file}
    ? (ask_developer("Repo-root markdown filename? (example: notes.md; subpaths not allowed)") → requested_file
       | requested_file ∃ ∨ halt("Requested output mode requires a repo-root markdown filename")
       | invoke(gybis-arch-describe_output_path_guard(requested_file)) → true
       | return(mode_selected = true ∧ mode_selected_explicit = true ∧ output_path = requested_file))
    : return(mode_selected = true ∧ mode_selected_explicit = true)

λ gybis-arch-describe_output_target(mode).
  mode = response_only → return(output_target_resolved = true ∧ output_path = none)
  | mode ∈ {prompted_file_only, response_and_prompted_file} → return(output_target_resolved = true ∧ output_path = requested_file)
  | mode = default_file_only → return(output_target_resolved = true ∧ output_path = "arch-describe.md")
  | mode = response_and_default_file → return(output_target_resolved = true ∧ output_path = "arch-describe.md")
  | ¬mode_selected_explicit → halt("Cannot resolve output target without explicit human mode selection")

λ gybis-arch-describe_output_path_guard(path).
  path_matches(path, *.md) ∧ ¬contains(path, "/") ∧ ¬contains(path, "\\") ∧ ¬contains(path, "..")
    → true
  | ¬path_matches(path, *.md) ∨ contains(path, "/") ∨ contains(path, "\\") ∨ contains(path, "..")
    → halt("Output path must be a repo-root markdown filename")

λ gybis-arch-describe_overwrite_guard(path).
  verify(path ∃)
    ? (ask_developer("File " ⊕ path ⊕ " exists. Overwrite? [yes/no]") → overwrite_choice
       | overwrite_choice = yes ∨ halt("Output file overwrite not approved"))
    : true

λ gybis-arch-describe_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING} → allow(read(path))
  | state = DELIVERING
    → allow(read(path)) ∧ allow(write(path)) only_if(path = output_path ∧ path_matches(path, *.md))
  | ¬(state ∈ {STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING}) → deny(write(path))

λ gybis-arch-describe_output_dispatch(prose, mode, output_path).
  require(mode_selected_explicit = true) ∨ halt("Output dispatch blocked: explicit human mode selection missing")
  mode = response_only → output(AI_response, prose)
  | mode ∈ {prompted_file_only, default_file_only}
    ? (invoke(gybis-arch-describe_overwrite_guard(output_path)) → true
       | write(output_path, prose)
       | output("Saved markdown to " ⊕ output_path))
  | mode ∈ {response_and_prompted_file, response_and_default_file}
    ? (invoke(gybis-arch-describe_overwrite_guard(output_path)) → true
       | write(output_path, prose)
       | output(AI_response, prose)
       | output("Saved markdown to " ⊕ output_path))

λ gybis-arch-describe_protocol_evidence(x).
  output_manifest ≡ {
    mode_selected_by_human: selected_mode,
    mode_selected_explicit: true,
    filename_prompted: selected_mode ∈ {prompted_file_only, response_and_prompted_file},
    output_path: output_path,
    sources_read: [architecture.md],
    architecture_scope_verified: true,
    startup_checks_passed: true
  }
  | emit(output_manifest) → protocol_evidence_emitted = true

λ gybis-arch-describe_orientation_output(x).
  report_orientation: selected_orientation ∈ {FP, OOP} ∨ unknown("missing S1.paradigm_preference")
  | orientation_summary: describe_only_if_present_in_architecture
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

λ gybis-arch-describe_verify_output_scope(prose).
  required_signals:
    - includes_S5_S4_S3_S2_S1_plain_english = true
    - architecture_relationship_focused = true
    - architecture_only_grounding = true
  | forbidden_signals:
    - mentions(specs/**/*.allium)
    - mentions(src/**)
    - mentions(tests/**)
    - contains_code_fence
    - contains_implementation_examples
    - contains_test_examples
    - introduces_claim_without_architecture_evidence
  | all(required_signals) ∧ none(forbidden_signals) → return(architecture_scope_verified = true)
  | otherwise → halt("Generated architecture description drifted beyond architecture.md-only scope")

λ gybis-arch-describe_output_constraints(x).
  ¬lambda_notation ∧ ¬syntax_output
  | plain_english(product_manager) | business_vocabulary ∧ concrete_examples
  | ¬invent(¬exists(architecture.md)) | only describe what architecture.md contains
  | flag(gap ∨ empty ∨ underdeveloped) ∧ ¬speculate | highlight unknowns without guessing
  | include(orientation_output: selected_orientation_or_unknown ∧ explicit_gaps)
  | ¬reference(specs/**/*.allium ∨ src/** ∨ tests/**) in generated_content
  | ¬emit(code_fences ∨ implementation_examples ∨ test_examples)
  | ¬modify(architecture.md ∨ specs/**/*.allium ∨ internal/reference/**)
  | write_only(repo_root_markdown_filename = output_path) | ¬write(subpaths ∨ non_markdown)
  | explicit_human_output_selection_required: true | ¬implicit_default_progression
  | protocol_evidence_required_before_complete: true
  | mode = response_only → output → AI_response ∧ ¬file
  | mode ∈ {prompted_file_only, default_file_only} → output → markdown_file ∧ status_response
  | mode ∈ {response_and_prompted_file, response_and_default_file} → output → AI_response ∧ markdown_file

λ gybis-arch-describe_audience(x).
  understand(business_context) ∧ ¬know(system_specific)
  | expert(vision ∧ goals ∧ business ∧ constraints)
  | zero_prior_knowledge(technical_architecture ∧ vsm)
  | needs: what_system_is ∧ why_it_matters ∧ what_drives_decisions

λ gybis-arch-describe_boundary(¬).
  ¬create_specs ∧ ¬modify(architecture.md) ∧ ¬modify_allium_ref
  | writes_limited_to(repo_root_markdown_filename)