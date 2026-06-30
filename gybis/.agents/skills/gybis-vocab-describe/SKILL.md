---
name: gybis-vocab-describe
description: Use for `/gybis-vocab-describe` or `/gv-describe`.
---

λ gybis-vocab-describe(x).
  purpose: Document the shared canonical term set (DDD ubiquitous language) in plain English prose for non-technical stakeholders (product managers, business analysts, domain experts)
  | input: vocabulary.md (exists ∧ complete)
  | output: Plain English prose describing core domain concepts and their relationships
  | mode: mixed (AI + human output selection)
  | gate: vocabulary.md ∃

λ gybis-vocab-describe_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | read(vocabulary.md) → content
  | parse(content) → vocabulary_terms
  | precondition: vocabulary_terms ∃ ∧ count > 0
  | transition(INIT → MODE_SELECTION)

λ gybis-vocab-describe_mode(m).
  valid_modes: {response_only, prompted_file_only, default_file_only, response_and_prompted_file, response_and_default_file}
  | default: response_only

λ gybis-vocab-describe_mode_gate(state, mode).
  state = INIT ∧ mode ∈ valid_modes → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ valid_modes) → halt("Invalid output mode selection")

λ gybis-vocab-describe_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTION, MODE_SELECTED, STARTUP_CHECKS, RESOLVING_OUTPUT, GENERATING, DELIVERING, COMPLETE}
  | transition(INIT → MODE_SELECTION) always
  | transition(MODE_SELECTION → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(mode_selected = true)
  | transition(STARTUP_CHECKS → RESOLVING_OUTPUT) only_if(startup_checks = true)
  | transition(RESOLVING_OUTPUT → GENERATING) only_if(output_target_resolved = true)
  | transition(GENERATING → DELIVERING) only_if(final_prose ∃)
  | transition(DELIVERING → COMPLETE) only_if(delivery_complete = true)

λ gybis-vocab-describe_output_selection(x).
  ask_developer("Output mode? [response_only/prompted_file_only/default_file_only/response_and_prompted_file/response_and_default_file]") → selected_mode
  | selected_mode ∈ valid_modes ∨ halt("Output mode must be one of the supported options")
  | selected_mode ∈ {prompted_file_only, response_and_prompted_file}
    ? (ask_developer("Repo-root markdown filename? (example: vocab-describe.md; subpaths not allowed)") → requested_file
       | invoke(gybis-vocab-describe_output_path_guard(requested_file)) → true
       | return(mode_selected = true ∧ output_path = requested_file))
    : return(mode_selected = true)

λ gybis-vocab-describe_output_target(mode).
  mode = response_only → return(output_target_resolved = true ∧ output_path = none)
  | mode ∈ {prompted_file_only, response_and_prompted_file} → return(output_target_resolved = true ∧ output_path = requested_file)
  | mode = default_file_only → return(output_target_resolved = true ∧ output_path = "vocab-describe.md")
  | mode = response_and_default_file → return(output_target_resolved = true ∧ output_path = "vocab-describe.md")

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

λ gybis-vocab-describe_deliver(prose, output_mode, output_path).
  output_mode = response_only
    ? return(response ≔ prose)
    : (output_mode ∈ {prompted_file_only, default_file_only, response_and_prompted_file, response_and_default_file}
       ? (write(output_path, prose) → persisted
          | return(file_written = true ∧ response = prose))
       : halt("Invalid output mode"))
