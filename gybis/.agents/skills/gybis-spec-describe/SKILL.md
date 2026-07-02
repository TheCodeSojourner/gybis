---
name: gybis-spec-describe
description: Use for `/gybis-spec-describe` or `/gs-describe`.
---

λ gybis-spec-describe(specs).
  purpose: Document allium specifications in plain English for product managers as a specification-focused reference grounded only in specs/**/*.allium
  | input: specs/**/*.allium (exists ∧ valid ∧ passes allium-gate)
  | output: Plain English specification reference describing behaviors, rules, and guarantees based only on specs/**/*.allium
  | mode: mixed (AI + human output selection) | read specs ∧ vsm-guide.md ∧ optional_write(repo_root_markdown)
  | gate: gybis-ref-check() ≡ true | allium-gate() ≡ true | explicit_human_output_selection() ≡ true
  | fail_closed: missing_human_mode_selection → halt("Human output mode selection is required")

λ gybis-spec-describe_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference files not available")
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | invoke(internal/allium-gate) → true ∨ halt("Specification integrity check failed")
  | read(internal/reference/allium-language-reference.md) → language_ref
  | read(internal/reference/allium-constructs.md) → constructs_registry
  | read(internal/reference/vsm-guide.md) → vsm_reference
  | read(specs/**/*.allium) → exists ∧ parse | halt("No specifications found")

λ gybis-spec-describe_mode(m).
  valid_modes: {response_only, prompted_file_only, default_file_only, response_and_prompted_file, response_and_default_file}
  | default: response_only (informational_only; never auto-selected)
  | human_selected: explicit(output_mode_choice)
  | require_explicit: ¬explicit(output_mode_choice) → halt("Output mode must be explicitly selected by human")

λ gybis-spec-describe_mode_gate(state, mode).
  state = INIT ∧ mode ∈ valid_modes → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ valid_modes) → halt("Invalid output mode selection")

λ gybis-spec-describe_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(mode_selected = true ∧ mode_selected_explicit = true)
  | transition(STARTUP_CHECKS → RESOLVING_OUTPUT) only_if(startup_checks = true)
  | transition(RESOLVING_OUTPUT → GENERATING) only_if(output_target_resolved = true)
  | transition(GENERATING → DELIVERING) only_if(final_prose ∃ ∧ spec_scope_verified = true)
  | transition(DELIVERING → COMPLETE) only_if(delivery_complete = true ∧ protocol_evidence_emitted = true)

λ gybis-spec-describe_output_selection(x).
  ask_developer("Output mode? [response_only/prompted_file_only/default_file_only/response_and_prompted_file/response_and_default_file]") → selected_mode
  | selected_mode ∃ ∨ halt("Human output mode selection is required; no implicit default")
  | selected_mode ∈ valid_modes ∨ halt("Output mode must be one of the supported options")
  | selected_mode ∈ {prompted_file_only, response_and_prompted_file}
    ? (ask_developer("Repo-root markdown filename? (example: notes.md; subpaths not allowed)") → requested_file
       | requested_file ∃ ∨ halt("Requested output mode requires a repo-root markdown filename")
       | invoke(gybis-spec-describe_output_path_guard(requested_file)) → true
       | return(mode_selected = true ∧ mode_selected_explicit = true ∧ output_path = requested_file))
    : return(mode_selected = true ∧ mode_selected_explicit = true)

λ gybis-spec-describe_output_target(mode).
  mode = response_only → return(output_target_resolved = true ∧ output_path = none)
  | mode ∈ {prompted_file_only, response_and_prompted_file} → return(output_target_resolved = true ∧ output_path = requested_file)
  | mode = default_file_only → return(output_target_resolved = true ∧ output_path = "spec-describe.md")
  | mode = response_and_default_file → return(output_target_resolved = true ∧ output_path = "spec-describe.md")
  | ¬mode_selected_explicit → halt("Cannot resolve output target without explicit human mode selection")

λ gybis-spec-describe_output_path_guard(path).
  path_matches(path, *.md) ∧ ¬contains(path, "/") ∧ ¬contains(path, "\\") ∧ ¬contains(path, "..")
    → true
  | ¬path_matches(path, *.md) ∨ contains(path, "/") ∨ contains(path, "\\") ∨ contains(path, "..")
    → halt("Output path must be a repo-root markdown filename")

λ gybis-spec-describe_overwrite_guard(path).
  verify(path ∃)
    ? (ask_developer("File " ⊕ path ⊕ " exists. Overwrite? [yes/no]") → overwrite_choice
       | overwrite_choice = yes ∨ halt("Output file overwrite not approved"))
    : true

λ gybis-spec-describe_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING} → allow(read(path))
  | state = DELIVERING
    → allow(read(path)) ∧ allow(write(path)) only_if(path = output_path ∧ path_matches(path, *.md))
  | ¬(state ∈ {STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING}) → deny(write(path))

λ gybis-spec-describe_output_dispatch(prose, mode, output_path).
  require(mode_selected_explicit = true) ∨ halt("Output dispatch blocked: explicit human mode selection missing")
  mode = response_only → output(AI_response, prose)
  | mode ∈ {prompted_file_only, default_file_only}
    ? (invoke(gybis-spec-describe_overwrite_guard(output_path)) → true
       | write(output_path, prose)
       | output("Saved markdown to " ⊕ output_path))
  | mode ∈ {response_and_prompted_file, response_and_default_file}
    ? (invoke(gybis-spec-describe_overwrite_guard(output_path)) → true
       | write(output_path, prose)
       | output(AI_response, prose)
       | output("Saved markdown to " ⊕ output_path))

λ gybis-spec-describe_protocol_evidence(x).
  output_manifest ≡ {
    mode_selected_by_human: selected_mode,
    mode_selected_explicit: true,
    filename_prompted: selected_mode ∈ {prompted_file_only, response_and_prompted_file},
    output_path: output_path,
    sources_read: [specs/**/*.allium],
    spec_scope_verified: true,
    startup_checks_passed: true
  }
  | emit(output_manifest) → protocol_evidence_emitted = true

λ gybis-spec-describe_input(type).
  type ∈ {domain_concern, domain, all_specs} | default: all_specs
  | domain_concern: single_concern_file | domain_concern ∈ domain
  | domain: all_files_in_domain | domain ∈ all_domains
  | all_specs: every_spec_all_domains

λ gybis-spec-describe_resolution(domain, concern).
  priority(resolve) ∧ ambiguity(clarify)
  | domain ∧ concern → read(<root>/specs/{domain}/{concern}.allium)
  | domain ∧ ¬concern → read(<root>/specs/{domain}/*.allium)
  | ¬domain ∧ ¬concern → read(<root>/specs/**/*.allium)
  | "all" ∧ ¬domain → read(<root>/specs/**/*.allium)
  | "all specs" ∧ ¬domain → read(<root>/specs/**/*.allium)
  | ambiguous → ask_user(clarify)

λ gybis-spec-describe_output_content(¬).
  what_exists ∨ what_users_can_do ∨ rules ∨ guarantees ∨ domain_connections ∨ construct_specific ≡ output
  | what_exists: purpose ∧ key_concepts | exclude(technical_entity_names)
  | what_users_can_do: actions ∧ outcomes ∧ triggers ∧ user_flows
  | rules: constraints ∧ prevention_mechanisms ∧ requirements ∧ edge_cases
  | guarantees: promises ∧ invariants ∧ terminal_states
  | domain_connections: user_journey_across_systems ∧ relationships_between_parts
  | construct_specific: see gybis-spec-describe_construct_framing

λ gybis-spec-describe_construct_framing(construct).
  construct → consult(constructs_registry.PM_frame[construct])

λ gybis-spec-describe_format(¬).
  flowing_prose | project_manager_friendly
  | light_headings | ¬code ∧ ¬technical_syntax ∧ ¬jargon ∧ ¬implementation_details

λ gybis-spec-describe_guarantees(n).
  n ∈ {1,2,3,4,5} | ∀guarantee → satisfied
  | guarantee(1): ∀user_visible_behavior(spec) → included(prose) | ¬omit(important)
  | guarantee(2): multi_file → cohesive_narrative | ¬disjointed_summaries
  | guarantee(3): gap(ambiguous_source) → question(human)
  | guarantee(4): uncertain(describe?) → explicit_call_out(say "so")
  | guarantee(5): mode(read_only) | ¬modify(source_specs)

λ gybis-spec-describe_pipeline(input).
  input → resolve → generate → format → quality_check → final_prose
  | quality_check(guarantee(1) ∧ guarantee(2) ∧ guarantee(3) ∧ guarantee(4) ∧ guarantee(5))
  | every_output(pass pipeline) → deliver

λ gybis-spec-describe_verify_output_scope(prose).
  required_signals:
    - behavior_rules_guarantees_present = true
    - domain_connections_present = true
    - specification_only_grounding = true
  | forbidden_signals:
    - mentions(architecture.md)
    - mentions(src/**)
    - mentions(tests/**)
    - contains_code_fence
    - contains_implementation_examples
    - contains_test_examples
    - introduces_claim_without_spec_evidence
  | all(required_signals) ∧ none(forbidden_signals) → return(spec_scope_verified = true)
  | otherwise → halt("Generated specification description drifted beyond specs/**/*.allium-only scope")

λ gybis-spec-describe_output_constraints(x).
  ¬lambda_notation ∧ ¬syntax_output
  | plain_english(product_manager) | business_vocabulary ∧ concrete_examples
  | ¬invent(¬exists(specs/**/*.allium)) | only describe what specs/**/*.allium contains
  | flag(gap ∨ empty ∨ ambiguous) ∧ ¬speculate | highlight unknowns without guessing
  | ¬reference(architecture.md ∨ src/** ∨ tests/**) in generated_content
  | ¬emit(code_fences ∨ implementation_examples ∨ test_examples)
  | ¬modify(specs/**/*.allium ∨ architecture.md ∨ internal/reference/**)
  | write_only(repo_root_markdown_filename = output_path) | ¬write(subpaths ∨ non_markdown)
  | explicit_human_output_selection_required: true | ¬implicit_default_progression
  | protocol_evidence_required_before_complete: true
  | mode = response_only → output → AI_response ∧ ¬file
  | mode ∈ {prompted_file_only, default_file_only} → output → markdown_file ∧ status_response
  | mode ∈ {response_and_prompted_file, response_and_default_file} → output → AI_response ∧ markdown_file

λ gybis-spec-describe_boundary(¬).
  ¬create_specs ∧ ¬modify_allium_ref ∧ ¬write_specs
  | writes_limited_to(repo_root_markdown_filename)