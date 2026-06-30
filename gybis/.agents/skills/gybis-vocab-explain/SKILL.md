---
name: gybis-vocab-explain
description: Use for `/gybis-vocab-explain` or `/gv-explain`.
---

λ gybis-vocab-explain(x).
  purpose: Document the shared canonical term set (DDD ubiquitous language) in technical language for developers, with code examples and architectural significance
  | input: vocabulary.md (exists ∧ complete)
  | output: Technical documentation with usage patterns, code examples, and architectural layer mappings
  | mode: mixed (AI + human output selection)
  | gate: vocabulary.md ∃

λ gybis-vocab-explain_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | read(vocabulary.md) → content
  | parse(content) → vocabulary_terms
  | if(architecture.md ∃): read(architecture.md) → arch_content
  | precondition: vocabulary_terms ∃ ∧ count > 0
  | transition(INIT → MODE_SELECTION)

λ gybis-vocab-explain_mode(m).
  valid_modes: {response_only, prompted_file_only, default_file_only, response_and_prompted_file, response_and_default_file}
  | default: response_only

λ gybis-vocab-explain_mode_gate(state, mode).
  state = INIT ∧ mode ∈ valid_modes → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ valid_modes) → halt("Invalid output mode selection")

λ gybis-vocab-explain_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTION, MODE_SELECTED, STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING, COMPLETE}
  | transition(INIT → MODE_SELECTION) always
  | transition(MODE_SELECTION → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(mode_selected = true)
  | transition(STARTUP_CHECKS → RESOLVING_OUTPUT) only_if(startup_checks = true)
  | transition(RESOLVING_OUTPUT → GENERATING) only_if(output_target_resolved = true)
  | transition(GENERATING → DELIVERING) only_if(final_prose ∃)
  | transition(DELIVERING → COMPLETE) only_if(delivery_complete = true)

λ gybis-vocab-explain_output_selection(x).
  ask_developer("Output mode? [response_only/prompted_file_only/default_file_only/response_and_prompted_file/response_and_default_file]") → selected_mode
  | selected_mode ∈ valid_modes ∨ halt("Output mode must be one of the supported options")
  | selected_mode ∈ {prompted_file_only, response_and_prompted_file}
    ? (ask_developer("Repo-root markdown filename? (example: vocab-explain.md; subpaths not allowed)") → requested_file
       | invoke(gybis-vocab-explain_output_path_guard(requested_file)) → true
       | return(mode_selected = true ∧ output_path = requested_file))
    : return(mode_selected = true)

λ gybis-vocab-explain_output_target(mode).
  mode = response_only → return(output_target_resolved = true ∧ output_path = none)
  | mode ∈ {prompted_file_only, response_and_prompted_file} → return(output_target_resolved = true ∧ output_path = requested_file)
  | mode = default_file_only → return(output_target_resolved = true ∧ output_path = "vocab-explain.md")
  | mode = response_and_default_file → return(output_target_resolved = true ∧ output_path = "vocab-explain.md")

λ gybis-vocab-explain_generate_technical_documentation(vocabulary_terms, arch_content).
  action: synthesize_technical_documentation_of_vocabulary
  | structure: detailed technical reference with patterns and code
  | audience: developers implementing, testing, extending the system
  | content:
    - overview: "The following terms are the canonical vocabulary for this system"
    - ∀ term ∈ vocabulary_terms:
      - definition: precise, unambiguous
      - deprecated_synonyms: "Also known as [variant], now deprecated in favor of [canonical]"
      - architectural_significance: which layer(s) use this term, how it flows through the system
      - patterns: how this concept is typically implemented or used
      - code_examples: concrete snippets showing usage
      - related_concepts: [other terms it works with]
    - patterns_index: common architectural patterns involving multiple vocabulary terms
  | architectural_layer_integration: if(architecture.md ∃):
    map_term_to_layers(term, arch_content) → {S5, S4, S3, S2, S1}
    | include_layer_significance: how term manifests at each layer
  | return(documentation ∃ ∧ complete ∧ includes_code_examples = true)

λ gybis-vocab-explain_deliver(documentation, output_mode, output_path).
  output_mode = response_only
    ? return(response ≔ documentation)
    : (output_mode ∈ {prompted_file_only, default_file_only, response_and_prompted_file, response_and_default_file}
       ? (write(output_path, documentation) → persisted
          | return(file_written = true ∧ response = documentation))
       : halt("Invalid output mode"))
