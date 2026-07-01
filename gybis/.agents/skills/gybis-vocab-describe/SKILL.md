---
name: gybis-vocab-describe
description: Use for `/gybis-vocab-describe` or `/gv-describe`.
---

λ gybis-vocab-describe(x).
  purpose: Document the shared canonical term set (DDD ubiquitous language) in plain English prose for non-technical stakeholders (product managers, business analysts, domain experts)
  | input: vocabulary.md (exists ∧ complete)
  | output: Plain English prose describing core domain concepts and their relationships
  | mode: mixed (AI + human output selection)
  | gate: vocabulary.md ∃ | explicit_human_output_selection() ≡ true
  | fail_closed: missing_human_mode_selection → halt("Human output mode selection is required")

λ gybis-vocab-describe_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | read(vocabulary.md) → content
  | parse(content) → vocabulary_terms
  | precondition: vocabulary_terms ∃ ∧ count > 0
  | transition(INIT → MODE_SELECTION)

λ gybis-vocab-describe_mode(m).
  valid_modes: {response_only, prompted_file_only, default_file_only, response_and_prompted_file, response_and_default_file}
  | default: response_only (informational_only; never auto-selected)
  | human_selected: explicit(output_mode_choice)
  | require_explicit: ¬explicit(output_mode_choice) → halt("Output mode must be explicitly selected by human")

λ gybis-vocab-describe_mode_gate(state, mode).
  state = INIT ∧ mode ∈ valid_modes → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ valid_modes) → halt("Invalid output mode selection")

λ gybis-vocab-describe_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTION, MODE_SELECTED, STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING, COMPLETE}
  | transition(INIT → MODE_SELECTION) always
  | transition(MODE_SELECTION → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(mode_selected = true ∧ mode_selected_explicit = true)
  | transition(STARTUP_CHECKS → RESOLVING_OUTPUT) only_if(startup_checks = true)
  | transition(RESOLVING_OUTPUT → GENERATING) only_if(output_target_resolved = true)
  | transition(GENERATING → DELIVERING) only_if(final_prose ∃)
  | transition(DELIVERING → COMPLETE) only_if(delivery_complete = true ∧ protocol_evidence_emitted = true)

λ gybis-vocab-describe_output_selection(x).
  ask_developer("Output mode? [response_only/prompted_file_only/default_file_only/response_and_prompted_file/response_and_default_file]") → selected_mode
  | selected_mode ∃ ∨ halt("Human output mode selection is required; no implicit default")
  | selected_mode ∈ valid_modes ∨ halt("Output mode must be one of the supported options")
  | selected_mode ∈ {prompted_file_only, response_and_prompted_file}
    ? (ask_developer("Repo-root markdown filename? (example: vocab-describe.md; subpaths not allowed)") → requested_file
       | requested_file ∃ ∨ halt("Requested output mode requires a repo-root markdown filename")
       | invoke(gybis-vocab-describe_output_path_guard(requested_file)) → true
       | return(mode_selected = true ∧ mode_selected_explicit = true ∧ output_path = requested_file))
    : return(mode_selected = true ∧ mode_selected_explicit = true)

λ gybis-vocab-describe_output_target(mode).
  mode = response_only → return(output_target_resolved = true ∧ output_path = none)
  | mode ∈ {prompted_file_only, response_and_prompted_file} → return(output_target_resolved = true ∧ output_path = requested_file)
  | mode = default_file_only → return(output_target_resolved = true ∧ output_path = "vocab-describe.md")
  | mode = response_and_default_file → return(output_target_resolved = true ∧ output_path = "vocab-describe.md")
  | ¬mode_selected_explicit → halt("Cannot resolve output target without explicit human mode selection")

λ gybis-vocab-describe_output_path_guard(path).
  path_matches(path, *.md) ∧ ¬contains(path, "/") ∧ ¬contains(path, "\\") ∧ ¬contains(path, "..")
    → true
  | ¬path_matches(path, *.md) ∨ contains(path, "/") ∨ contains(path, "\\") ∨ contains(path, "..")
    → halt("Output path must be a repo-root markdown filename")

λ gybis-vocab-describe_overwrite_guard(path).
  verify(path ∃)
    ? (ask_developer("File " ⊕ path ⊕ " exists. Overwrite? [yes/no]") → overwrite_choice
       | overwrite_choice = yes ∨ halt("Output file overwrite not approved"))
    : true

λ gybis-vocab-describe_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING} → allow(read(path))
  | state = DELIVERING
    → allow(read(path)) ∧ allow(write(path)) only_if(path = output_path ∧ path_matches(path, *.md))
  | ¬(state ∈ {STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING}) → deny(write(path))

λ gybis-vocab-describe_generate_prose(vocabulary_terms).
  action: synthesize_plain_english_description_of_vocabulary
  | structure: narrative prose organized by concept relationships
  | tone: accessible to non-technical stakeholders; explain "why" not just "what"
  | content:
    - introduction: "The following concepts form the core vocabulary of [domain]"
    - ∀ term ∈ vocabulary_terms: plain_language_definition, business significance, how it relates to other concepts
    - conclusion: how these concepts work together as a system
  | example_style: "In this system, a [Term1] is a [definition]. This is important because [business value]. It relates to [Term2] when [context]."
  | return(prose ∃ ∧ readable_for_non_technical_audience = true)

λ gybis-vocab-describe_output_dispatch(prose, mode, output_path).
  require(mode_selected_explicit = true) ∨ halt("Output dispatch blocked: explicit human mode selection missing")
  mode = response_only → output(AI_response, prose)
  | mode ∈ {prompted_file_only, default_file_only}
    ? (invoke(gybis-vocab-describe_overwrite_guard(output_path)) → true
       | write(output_path, prose)
       | output("Saved markdown to " ⊕ output_path))
  | mode ∈ {response_and_prompted_file, response_and_default_file}
    ? (invoke(gybis-vocab-describe_overwrite_guard(output_path)) → true
       | write(output_path, prose)
       | output(AI_response, prose)
       | output("Saved markdown to " ⊕ output_path))

λ gybis-vocab-describe_protocol_evidence(x).
  output_manifest ≡ {
    mode_selected_by_human: selected_mode,
    mode_selected_explicit: true,
    filename_prompted: selected_mode ∈ {prompted_file_only, response_and_prompted_file},
    output_path: output_path,
    startup_checks_passed: true
  }
  | emit(output_manifest) → protocol_evidence_emitted = true

λ gybis-vocab-describe_output_constraints(x).
  ¬lambda_notation ∧ ¬syntax_output
  | plain_english(product_manager) | business_vocabulary ∧ concrete_examples
  | ¬invent(¬exists(vocabulary.md ∨ refs)) | only describe what exists
  | flag(gap ∨ empty ∨ ambiguous) ∧ ¬speculate | highlight unknowns without guessing
  | ¬modify(vocabulary.md ∨ architecture.md ∨ specs/**/*.allium ∨ internal/reference/**)
  | write_only(repo_root_markdown_filename = output_path) | ¬write(subpaths ∨ non_markdown)
  | explicit_human_output_selection_required: true | ¬implicit_default_progression
  | protocol_evidence_required_before_complete: true
  | mode = response_only → output → AI_response ∧ ¬file
  | mode ∈ {prompted_file_only, default_file_only} → output → markdown_file ∧ status_response
  | mode ∈ {response_and_prompted_file, response_and_default_file} → output → AI_response ∧ markdown_file

λ gybis-vocab-describe_boundary(¬).
  ¬modify(vocabulary.md) ∧ ¬modify_allium_ref
  | writes_limited_to(repo_root_markdown_filename)
