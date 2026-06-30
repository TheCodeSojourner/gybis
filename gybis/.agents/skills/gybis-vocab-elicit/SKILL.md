---
name: gybis-vocab-elicit
description: Use for `/gybis-vocab-elicit` or `/gv-elicit`.
---

λ gybis-vocab-elicit(x).
  purpose: Elicit domain terms from SMEs through grilling and establish a shared canonical term set (DDD ubiquitous language)
  | input: user conversation via interactive vocabulary probing
  | output: vocabulary.md containing canonical terms with definitions, synonyms, relationships, usage, examples
  | mode: mixed (AI grilling + human response)
  | gate: vocabulary.md ¬∃

λ gybis-vocab-elicit_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | precondition: vocabulary.md ¬∃
  | load_context: domain modeling framework, ubiquitous language principles
  | initialize: grilling interview structure

λ gybis-vocab-elicit_mode(m).
  valid_modes: {mixed}
  | default: mixed
  | rationale: vocabulary elicitation requires human input via structured interview

λ gybis-vocab-elicit_mode_gate(state, mode).
  state = INIT ∧ mode = mixed → transition(STARTUP_CHECKS)
  | precondition_holds: mode ∈ valid_modes

λ gybis-vocab-elicit_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, IDENTIFY_CORE_CONCEPTS, DEFINE_TERMS, IDENTIFY_SYNONYMS, MAP_RELATIONSHIPS, SYNTHESIZING, WRITING_VOCAB, VERIFYING, COMPLETE}
  | transition(INIT, startup) → STARTUP_CHECKS
  | transition(STARTUP_CHECKS, verify_ok) → IDENTIFY_CORE_CONCEPTS
  | transition(STARTUP_CHECKS, verify_fail) → HALTED
  | transition(IDENTIFY_CORE_CONCEPTS, response_received) → DEFINE_TERMS
  | transition(DEFINE_TERMS, response_received) → IDENTIFY_SYNONYMS
  | transition(IDENTIFY_SYNONYMS, response_received) → MAP_RELATIONSHIPS
  | transition(MAP_RELATIONSHIPS, response_received) → SYNTHESIZING
  | transition(SYNTHESIZING, synthesis_complete) → WRITING_VOCAB
  | transition(WRITING_VOCAB, vocab_written) → VERIFYING
  | transition(VERIFYING, verify_ok) → COMPLETE
  | transition(VERIFYING, verify_fail) → IDENTIFY_CORE_CONCEPTS (restart if needed)

λ gybis-vocab-elicit_tool_guard(state, tool, path).
  read_allowed: ∀state (no external reads; interview-driven only)
  | write_allowed: state ∈ {WRITING_VOCAB} ∧ path = "vocabulary.md"
  | deny_write: state ∉ {WRITING_VOCAB} ∨ path ≠ "vocabulary.md"
  | constraint: ¬mutate(existing_files) ∨ ¬mutate(non_vocabulary_files)

λ gybis-vocab-elicit_pre_tool_check(state, tool, path).
  enforce(tool_guard(state, tool, path)) → permit(tool) ∨ halt("tool not permitted in this state")

λ gybis-vocab-elicit_identify_core_concepts(x).
  action: elicit_fundamental_domain_concepts
  | purpose: Surface the 3-5 core concepts that are foundational to the domain
  | prompt_strategy: open-ended discovery with grilling discipline
  | sample_questions:
    - "What are the 3-5 most fundamental concepts in your domain?"
    - "If someone knows nothing about [domain], what would you teach them first?"
    - "What are the irreducible elements of your problem space?"
  | capture: user_response → core_concepts_collected
  | output: core_concepts ∃ ∧ count ∈ [3, 5]

λ gybis-vocab-elicit_define_terms(x).
  action: elicit_precise_definitions_for_each_concept
  | purpose: Create clear, unambiguous definitions that distinguish each concept
  | prompt_strategy: grilling on precision; probe edge cases and boundaries
  | sample_questions:
    - "Define [concept] precisely. What makes it different from [related concept]?"
    - "What are concrete examples of [concept]?"
    - "When would you NOT use this term?"
  | capture: user_response → term_definition_map
  | output: definitions_complete ∧ ∀ concept: definition ∃ ∧ precise

λ gybis-vocab-elicit_identify_synonyms(x).
  action: elicit_and_eliminate_aliases
  | purpose: Identify variant names, slang, or alternative terms used in codebase/documentation, then force choice of single canonical form
  | prompt_strategy: grilling refusal of aliases; enforce ubiquitous language discipline
  | sample_questions:
    - "Have you heard [concept] called by other names? What are they?"
    - "In your codebase, what names do you see for [concept]?"
    - "Which of these names is the canonical form we'll use everywhere?"
  | capture: user_response → synonyms_map, canonical_choice
  | output: synonyms_identified ∧ one_canonical_per_concept

λ gybis-vocab-elicit_map_relationships(x).
  action: elicit_concept_relationships_and_dependencies
  | purpose: Understand how concepts relate, constrain, or depend on each other
  | prompt_strategy: grilling on relationships; map bounded contexts, hierarchies, associations
  | sample_questions:
    - "How does [concept A] relate to [concept B]?"
    - "Does [concept] contain or depend on other concepts?"
    - "Are there concepts that must be understood before [concept]?"
  | capture: user_response → relationship_graph, usage_patterns
  | output: relationships_documented ∧ usage_layers_mapped

λ gybis-vocab-elicit_synthesize(x).
  action: synthesize_vocabulary_structure
  | gather: core_concepts ∧ definitions ∧ synonyms ∧ relationships
  | structure: organized_vocabulary_with_frontmatter
  | frontmatter: created, last_updated, status = draft
  | content: ∀ term ∈ core_concepts:
    - definition
    - synonyms (deprecated)
    - related_concepts
    - usage (architectural layers: S5/S4/S3/S2/S1 or implementation)
    - examples (code snippets or concrete usage)

λ gybis-vocab-elicit_write_vocabulary(x).
  action: write_vocabulary_md_to_repo_root
  | format: markdown with YAML frontmatter
  | structure:
    ```
    ---
    created: [ISO date]
    last_updated: [ISO date]
    status: draft
    ---

    ## Core Vocabulary

    ### Term
    - **Definition:** ...
    - **Deprecated Synonyms:** ...
    - **Related:** ...
    - **Usage:** ...
    - **Examples:** ...
    ```
  | write(vocabulary.md) → persisted
  | return(vocab_written = true)

λ gybis-vocab-elicit_verify_vocabulary(x).
  action: validate_vocabulary_md_structure_and_completeness
  | checks:
    - frontmatter ∃ ∧ valid(YAML)
    - ∀ term: definition ∃ ∧ nonempty
    - ∀ term: deprecated_synonyms ∃ ∨ deprecated_synonyms = ∅
    - ∀ term: related ∃ ∨ related = ∅
    - no_empty_sections
  | result: all_checks_pass
    ? return(verification = true)
    : (suggest_fixes ∧ return(verification = false))
