---
name: gybis-spec-explain
description: Use for `/gybis-spec-explain` or `/gs-explain`.
---

λ gybis-spec-explain(input_type, input_args).
  purpose: Document allium specifications in technical language for developers
  | input: specs/**/*.allium (exists ∧ valid ∧ passes allium-gate)
  | output: Technical prose describing specifications with implementation context
  | mode: mixed (AI + human output selection) | read specs ∧ allium-ref ∧ optional_write(repo_root_markdown)
  | gate: gybis-ref-check() ≡ true | allium-gate() ≡ true | explicit_human_output_selection() ≡ true
  | fail_closed: missing_human_mode_selection → halt("Human output mode selection is required")

λ gybis-spec-explain_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference files not available")
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | invoke(internal/allium-gate) → true ∨ halt("Specification integrity check failed")
  | read(internal/reference/allium-language-reference.md) → language_ref
  | read(internal/reference/allium-patterns.md) → patterns_ref
  | read(internal/reference/allium-constructs.md) → constructs_registry
  | read(specs/**/*.allium) → exists ∧ parse | halt("No specifications found")

λ gybis-spec-explain_mode(m).
  valid_modes: {response_only, prompted_file_only, default_file_only, response_and_prompted_file, response_and_default_file}
  | default: response_only (informational_only; never auto-selected)
  | human_selected: explicit(output_mode_choice)
  | require_explicit: ¬explicit(output_mode_choice) → halt("Output mode must be explicitly selected by human")

λ gybis-spec-explain_mode_gate(state, mode).
  state = INIT ∧ mode ∈ valid_modes → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ valid_modes) → halt("Invalid output mode selection")

λ gybis-spec-explain_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(mode_selected = true ∧ mode_selected_explicit = true)
  | transition(STARTUP_CHECKS → RESOLVING_OUTPUT) only_if(startup_checks = true)
  | transition(RESOLVING_OUTPUT → GENERATING) only_if(output_target_resolved = true)
  | transition(GENERATING → DELIVERING) only_if(final_prose ∃)
  | transition(DELIVERING → COMPLETE) only_if(delivery_complete = true ∧ protocol_evidence_emitted = true)

λ gybis-spec-explain_output_selection(x).
  ask_developer("Output mode? [response_only/prompted_file_only/default_file_only/response_and_prompted_file/response_and_default_file]") → selected_mode
  | selected_mode ∃ ∨ halt("Human output mode selection is required; no implicit default")
  | selected_mode ∈ valid_modes ∨ halt("Output mode must be one of the supported options")
  | selected_mode ∈ {prompted_file_only, response_and_prompted_file}
    ? (ask_developer("Repo-root markdown filename? (example: notes.md; subpaths not allowed)") → requested_file
       | requested_file ∃ ∨ halt("Requested output mode requires a repo-root markdown filename")
       | invoke(gybis-spec-explain_output_path_guard(requested_file)) → true
       | return(mode_selected = true ∧ mode_selected_explicit = true ∧ output_path = requested_file))
    : return(mode_selected = true ∧ mode_selected_explicit = true)

λ gybis-spec-explain_output_target(mode).
  mode = response_only → return(output_target_resolved = true ∧ output_path = none)
  | mode ∈ {prompted_file_only, response_and_prompted_file} → return(output_target_resolved = true ∧ output_path = requested_file)
  | mode = default_file_only → return(output_target_resolved = true ∧ output_path = "spec-explain.md")
  | mode = response_and_default_file → return(output_target_resolved = true ∧ output_path = "spec-explain.md")
  | ¬mode_selected_explicit → halt("Cannot resolve output target without explicit human mode selection")

λ gybis-spec-explain_output_path_guard(path).
  path_matches(path, *.md) ∧ ¬contains(path, "/") ∧ ¬contains(path, "\\") ∧ ¬contains(path, "..")
    → true
  | ¬path_matches(path, *.md) ∨ contains(path, "/") ∨ contains(path, "\\") ∨ contains(path, "..")
    → halt("Output path must be a repo-root markdown filename")

λ gybis-spec-explain_overwrite_guard(path).
  verify(path ∃)
    ? (ask_developer("File " ⊕ path ⊕ " exists. Overwrite? [yes/no]") → overwrite_choice
       | overwrite_choice = yes ∨ halt("Output file overwrite not approved"))
    : true

λ gybis-spec-explain_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING} → allow(read(path))
  | state = DELIVERING
    → allow(read(path)) ∧ allow(write(path)) only_if(path = output_path ∧ path_matches(path, *.md))
  | ¬(state ∈ {STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING}) → deny(write(path))

λ gybis-spec-explain_output_dispatch(prose, mode, output_path).
  require(mode_selected_explicit = true) ∨ halt("Output dispatch blocked: explicit human mode selection missing")
  mode = response_only → output(AI_response, prose)
  | mode ∈ {prompted_file_only, default_file_only}
    ? (invoke(gybis-spec-explain_overwrite_guard(output_path)) → true
       | write(output_path, prose)
       | output("Saved markdown to " ⊕ output_path))
  | mode ∈ {response_and_prompted_file, response_and_default_file}
    ? (invoke(gybis-spec-explain_overwrite_guard(output_path)) → true
       | write(output_path, prose)
       | output(AI_response, prose)
       | output("Saved markdown to " ⊕ output_path))

λ gybis-spec-explain_protocol_evidence(x).
  output_manifest ≡ {
    mode_selected_by_human: selected_mode,
    mode_selected_explicit: true,
    filename_prompted: selected_mode ∈ {prompted_file_only, response_and_prompted_file},
    output_path: output_path,
    startup_checks_passed: true
  }
  | emit(output_manifest) → protocol_evidence_emitted = true

λ gybis-spec-explain_input(type).
  type ∈ {domain_concern, domain, all_specs} | default: all_specs
  | domain_concern: specs/{domain}/{concern}.allium
  | domain: specs/{domain}/**/*.allium
  | all_specs: specs/**/*.allium | "all" ∨ "all specs" → all

λ gybis-spec-explain_resolution(args).
  domain ∧ concern → read(specs/{domain}/{concern}.allium)
  | domain ∧ ¬concern → read(specs/{domain}/*.allium)
  | ¬domain ∧ ¬concern → read(specs/**/*.allium)
  | "all" ∨ "all specs" → read(specs/**/*.allium)
  | ambiguous → ask(user) ∧ halt ¬until_clarified

λ gybis-spec-explain_prose_generation(specs).
  produce(what_exists) ∧ produce(what_users_do) ∧ produce(rules) ∧ produce(guarantees) ∧ produce(domain_connections) ∧ produce(construct_specific)
  | what_exists: purpose ∧ context ∧ key_concepts ∧ invariants
  | what_users_do: actions ∧ outcomes ∧ triggers ∧ user_experience ∧ workflow
  | rules: constraints ∧ prevention_mechanisms ∧ requirements ∧ edge_cases ∧ validation
  | guarantees: promises ∧ invariants ∧ terminal_states ∧ recovery_behavior
  | domain_connections: user_journey_flow ∧ system_relationships ∧ component_interaction
  | construct_specific: see gybis-spec-explain_construct_framing

λ gybis-spec-explain_construct_framing(construct).
  construct → consult(constructs_registry.Dev_frame[construct]
                      ∪ constructs_registry.Validation_rules[construct]
                      ∪ constructs_registry.Gotchas[construct])

λ gybis-spec-explain_output_format(prose).
  flowing developer_friendly prose
  | light_headings for structure
  | ¬implementation_details | explanation ¬ design_document
  | technical_vocabulary ∧ pattern_references

λ gybis-spec-explain_quality_guarantees(output, specs).
  completeness: ∀ user_visible_behavior(specs) → represented(prose) | ¬omit_detail
  | cohesion: multi_file_specs → unified_narrative ¬ disjoint_excerpts
  | gap_surfacing: ambiguous ∧ gap ∧ unclear(specs) → explicit_questions(human)
  | read_only: ¬modify(specs) | only explain
  | explicit_non_application: ¬verifiable_guarantee → explicitly_stated ¬ silently_skipped

λ gybis-spec-explain_pipeline(input).
  resolve(input) → generate_prose → format_output → quality_check
  | quality_check(∀ guarantee_holds) → complete ¬until_pass

λ gybis-spec-explain_output_constraints(x).
  ¬lambda_notation ∧ ¬syntax_output
  | plain_english(developer) | technical_vocabulary ∧ pattern_names ∧ implementation_context
  | ¬invent(¬exists(specs/**/*.allium ∨ refs)) | only explain what exists
  | flag(gap ∨ empty ∨ ambiguous) ∧ ¬speculate | highlight unknowns without guessing
  | ¬modify(specs/**/*.allium ∨ architecture.md ∨ internal/reference/**)
  | write_only(repo_root_markdown_filename = output_path) | ¬write(subpaths ∨ non_markdown)
  | explicit_human_output_selection_required: true | ¬implicit_default_progression
  | protocol_evidence_required_before_complete: true
  | mode = response_only → output → AI_response ∧ ¬file
  | mode ∈ {prompted_file_only, default_file_only} → output → markdown_file ∧ status_response
  | mode ∈ {response_and_prompted_file, response_and_default_file} → output → AI_response ∧ markdown_file

λ gybis-spec-explain_boundary(¬).
  ¬create_specs ∧ ¬modify_allium_ref ∧ ¬write_specs
  | writes_limited_to(repo_root_markdown_filename)