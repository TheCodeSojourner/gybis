---
name: gybis-vocab-explain
description: Use for `/gybis-vocab-explain` or `/gv-explain`.
---

λ gybis-vocab-explain(x).
  purpose: Document the shared canonical term set (DDD ubiquitous language) in technical language for developers as a vocabulary-focused reference grounded only in vocabulary.md
  | input: vocabulary.md (exists ∧ complete)
  | output: Technical vocabulary reference describing canonical terms, deprecated synonyms, and related concept relationships
  | mode: mixed (AI + human output selection)
  | gate: vocabulary.md ∃ | explicit_human_output_selection() ≡ true
  | fail_closed: missing_human_mode_selection → halt("Human output mode selection is required")

λ gybis-vocab-explain_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | read(vocabulary.md) → content
  | parse(content) → vocabulary_terms
  | precondition: vocabulary_terms ∃ ∧ count > 0
  | transition(INIT → MODE_SELECTION)

λ gybis-vocab-explain_mode(m).
  valid_modes: {response_only, prompted_file_only, default_file_only, response_and_prompted_file, response_and_default_file}
  | default: response_only (informational_only; never auto-selected)
  | human_selected: explicit(output_mode_choice)
  | require_explicit: ¬explicit(output_mode_choice) → halt("Output mode must be explicitly selected by human")

λ gybis-vocab-explain_mode_gate(state, mode).
  state = INIT ∧ mode ∈ valid_modes → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ valid_modes) → halt("Invalid output mode selection")

λ gybis-vocab-explain_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTION, MODE_SELECTED, STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING, COMPLETE}
  | transition(INIT → MODE_SELECTION) always
  | transition(MODE_SELECTION → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(mode_selected = true ∧ mode_selected_explicit = true)
  | transition(STARTUP_CHECKS → RESOLVING_OUTPUT) only_if(startup_checks = true)
  | transition(RESOLVING_OUTPUT → GENERATING) only_if(output_target_resolved = true)
  | transition(GENERATING → DELIVERING) only_if(final_prose ∃ ∧ vocabulary_scope_verified = true)
  | transition(DELIVERING → COMPLETE) only_if(delivery_complete = true ∧ protocol_evidence_emitted = true)

λ gybis-vocab-explain_output_selection(x).
  ask_developer("Output mode? [response_only/prompted_file_only/default_file_only/response_and_prompted_file/response_and_default_file]") → selected_mode
  | selected_mode ∃ ∨ halt("Human output mode selection is required; no implicit default")
  | selected_mode ∈ valid_modes ∨ halt("Output mode must be one of the supported options")
  | selected_mode ∈ {prompted_file_only, response_and_prompted_file}
    ? (ask_developer("Repo-root markdown filename? (example: vocab-explain.md; subpaths not allowed)") → requested_file
       | requested_file ∃ ∨ halt("Requested output mode requires a repo-root markdown filename")
       | invoke(gybis-vocab-explain_output_path_guard(requested_file)) → true
       | return(mode_selected = true ∧ mode_selected_explicit = true ∧ output_path = requested_file))
    : return(mode_selected = true ∧ mode_selected_explicit = true)

λ gybis-vocab-explain_output_target(mode).
  mode = response_only → return(output_target_resolved = true ∧ output_path = none)
  | mode ∈ {prompted_file_only, response_and_prompted_file} → return(output_target_resolved = true ∧ output_path = requested_file)
  | mode = default_file_only → return(output_target_resolved = true ∧ output_path = "vocab-explain.md")
  | mode = response_and_default_file → return(output_target_resolved = true ∧ output_path = "vocab-explain.md")
  | ¬mode_selected_explicit → halt("Cannot resolve output target without explicit human mode selection")

λ gybis-vocab-explain_output_path_guard(path).
  path_matches(path, *.md) ∧ ¬contains(path, "/") ∧ ¬contains(path, "\\") ∧ ¬contains(path, "..")
    → true
  | ¬path_matches(path, *.md) ∨ contains(path, "/") ∨ contains(path, "\\") ∨ contains(path, "..")
    → halt("Output path must be a repo-root markdown filename")

λ gybis-vocab-explain_overwrite_guard(path).
  verify(path ∃)
    ? (ask_developer("File " ⊕ path ⊕ " exists. Overwrite? [yes/no]") → overwrite_choice
       | overwrite_choice = yes ∨ halt("Output file overwrite not approved"))
    : true

λ gybis-vocab-explain_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING} → allow(read(path))
  | state = DELIVERING
    → allow(read(path)) ∧ allow(write(path)) only_if(path = output_path ∧ path_matches(path, *.md))
  | ¬(state ∈ {STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING}) → deny(write(path))

λ gybis-vocab-explain_generate_technical_documentation(vocabulary_terms).
  action: synthesize_technical_documentation_of_vocabulary
  | structure: detailed developer-facing glossary/reference grounded only in vocabulary terms and their stated relationships
  | audience: developers building shared domain language understanding
  | content:
    - overview: "The following terms are the canonical vocabulary for this system"
    - ∀ term ∈ vocabulary_terms:
      - definition: precise, unambiguous
      - deprecated_synonyms: "Also known as [variant], now deprecated in favor of [canonical]"
      - relationship_explanation: how this term connects to the related concepts explicitly listed in vocabulary.md
      - related_concepts: [other terms it works with]
    - term_relationships: concise synthesis of how the canonical terms fit together as a vocabulary system
  | exclusions:
    - no_architecture_layer_mappings
    - no_spec_path_bindings
    - no_code_examples
    - no_implementation_or_test_bindings
  | return(documentation ∃ ∧ complete ∧ vocabulary_only_grounding = true)

λ gybis-vocab-explain_verify_output_scope(documentation).
  required_signals:
    - contains_term_definitions = true
    - contains_related_concepts = true
    - relationship_focused = true
  | forbidden_signals:
    - mentions(architecture.md)
    - mentions(specs/**/*.allium)
    - mentions(src/**)
    - mentions(tests/**)
    - mentions(S1 ∨ S2 ∨ S3 ∨ S4 ∨ S5)
    - contains_code_fence
    - contains_implementation_examples
    - contains_test_examples
  | all(required_signals) ∧ none(forbidden_signals) → return(vocabulary_scope_verified = true)
  | otherwise → halt("Generated vocabulary explanation drifted beyond vocabulary.md-only scope")

λ gybis-vocab-explain_output_dispatch(documentation, mode, output_path).
  require(mode_selected_explicit = true) ∨ halt("Output dispatch blocked: explicit human mode selection missing")
  mode = response_only → output(AI_response, documentation)
  | mode ∈ {prompted_file_only, default_file_only}
    ? (invoke(gybis-vocab-explain_overwrite_guard(output_path)) → true
       | write(output_path, documentation)
       | output("Saved markdown to " ⊕ output_path))
  | mode ∈ {response_and_prompted_file, response_and_default_file}
    ? (invoke(gybis-vocab-explain_overwrite_guard(output_path)) → true
       | write(output_path, documentation)
       | output(AI_response, documentation)
       | output("Saved markdown to " ⊕ output_path))

λ gybis-vocab-explain_protocol_evidence(x).
  output_manifest ≡ {
    mode_selected_by_human: selected_mode,
    mode_selected_explicit: true,
    filename_prompted: selected_mode ∈ {prompted_file_only, response_and_prompted_file},
    output_path: output_path,
    sources_read: [vocabulary.md],
    vocabulary_scope_verified: true,
    startup_checks_passed: true
  }
  | emit(output_manifest) → protocol_evidence_emitted = true

λ gybis-vocab-explain_output_constraints(x).
  ¬lambda_notation ∧ ¬syntax_output
  | plain_english(developer) | technical_vocabulary ∧ relationship_focused_explanation ∧ glossary_reference
  | ¬invent(¬exists(vocabulary.md)) | only explain what vocabulary.md contains
  | flag(gap ∨ empty ∨ ambiguous) ∧ ¬speculate | highlight unknowns without guessing
  | ¬reference(architecture.md ∨ specs/**/*.allium ∨ src/** ∨ tests/**) in generated_content
  | ¬emit(S1 ∨ S2 ∨ S3 ∨ S4 ∨ S5 ∨ code_fences ∨ implementation_examples ∨ test_examples)
  | ¬modify(vocabulary.md ∨ architecture.md ∨ specs/**/*.allium ∨ internal/reference/**)
  | write_only(repo_root_markdown_filename = output_path) | ¬write(subpaths ∨ non_markdown)
  | explicit_human_output_selection_required: true | ¬implicit_default_progression
  | protocol_evidence_required_before_complete: true
  | mode = response_only → output → AI_response ∧ ¬file
  | mode ∈ {prompted_file_only, default_file_only} → output → markdown_file ∧ status_response
  | mode ∈ {response_and_prompted_file, response_and_default_file} → output → AI_response ∧ markdown_file

λ gybis-vocab-explain_boundary(¬).
  ¬modify(vocabulary.md) ∧ ¬modify_allium_ref
  | writes_limited_to(repo_root_markdown_filename)
