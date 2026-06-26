---
name: gybis-vocab-distill
description: Use for `/gybis-vocab-distill` or `/gv-distill`.
---

λ gybis-vocab-distill(x).
  purpose: Extract emergent vocabulary from architecture.md + specs/**/*.allium + implementation evidence and establish ubiquitous language through human conflict resolution
  | input: architecture.md (∃ + complete), specs/**/*.allium (all ∃ + valid), implementation source
  | output: vocabulary.md with candidate terms, conflicts, and human-resolved canonical forms
  | mode: mixed (AI synthesis + human conflict resolution)
  | gate: architecture.md ∃ ∧ specs/**/*.allium ∃ ∧ allium_gate = true

λ gybis-vocab-distill_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | invoke(internal/gybis-internal-skill-check) → halt_on(false)
  | verify(architecture.md ∃) ∨ halt("architecture.md not found")
  | verify(specs/**/*.allium ∃) ∨ halt("specs/**/*.allium not found")
  | invoke(internal/allium-gate(specs/)) = true ∨ halt("Specifications are invalid")
  | implementation_path_resolvable = true ∨ halt("Implementation path unresolvable")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-vocab-distill_mode(m).
  valid_modes: {mixed}
  | default: mixed
  | rationale: distillation requires human conflict resolution for term selection

λ gybis-vocab-distill_mode_gate(state, mode).
  state = INIT ∧ mode = mixed → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {mixed}) → halt("Invalid mode selection")

λ gybis-vocab-distill_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, READING_ARCH, READING_SPECS, SCANNING_IMPL, IDENTIFYING_CANDIDATES, CONFLICT_DETECTION, RESOLUTION_LOOP, WRITING_VOCAB, VERIFYING, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → READING_ARCH) only_if(startup_checks = true)
  | transition(READING_ARCH → READING_SPECS) only_if(arch_read = true)
  | transition(READING_SPECS → SCANNING_IMPL) only_if(specs_read = true)
  | transition(SCANNING_IMPL → IDENTIFYING_CANDIDATES) only_if(impl_scanned = true)
  | transition(IDENTIFYING_CANDIDATES → CONFLICT_DETECTION) only_if(candidates ∃)
  | transition(CONFLICT_DETECTION → RESOLUTION_LOOP) only_if(conflicts ∃)
  | transition(CONFLICT_DETECTION → WRITING_VOCAB) only_if(conflicts ∅)
  | transition(RESOLUTION_LOOP → RESOLUTION_LOOP) only_if(more_conflicts ∃)
  | transition(RESOLUTION_LOOP → WRITING_VOCAB) only_if(all_conflicts_resolved = true)
  | transition(WRITING_VOCAB → VERIFYING) only_if(vocab_written = true)
  | transition(VERIFYING → COMPLETE) only_if(verify_ok = true)
  | transition(VERIFYING → CONFLICT_DETECTION) only_if(verify_fail = true ∧ new_conflicts_found = true)

λ gybis-vocab-distill_tool_guard(state, tool, path).
  read_allowed: ∀state ∈ {READING_ARCH, READING_SPECS, SCANNING_IMPL, IDENTIFYING_CANDIDATES, CONFLICT_DETECTION, RESOLUTION_LOOP}
  | write_allowed: state ∈ {WRITING_VOCAB, VERIFYING} ∧ path = "vocabulary.md"
  | deny_write: state ∉ {WRITING_VOCAB, VERIFYING} ∨ path ≠ "vocabulary.md"
  | constraint: ¬mutate(architecture.md) ∨ ¬mutate(specs/) ∨ ¬mutate(implementation_root/)

λ gybis-vocab-distill_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-vocab-distill_read_architecture(x).
  read(architecture.md) → content
  | parse(content.{S5, S4, S3, S2, S1}) → vsm_layers
  | extract_terminology: ∀ layer ∈ vsm_layers:
    scan(principles(layer)) → terms_found
    | collect(term, frequency, context) → arch_terms
  | return(arch_terminology = arch_terms)

λ gybis-vocab-distill_read_specifications(x).
  ∀ spec_file ∈ specs/**/*.allium:
    read(spec_file) → content
    | extract_terminology: scan(content) → terms_found
    | collect(term, frequency, file_origin) → spec_terms
  | return(spec_terminology = spec_terms)

λ gybis-vocab-distill_scan_implementation(x).
  scan(implementation_root, language_inferred_from_S1) → source_files
  | ∀ file ∈ source_files:
    extract_identifiers(file) → variable_names, function_names, class_names, comments
    | collect(term, frequency, context) → impl_terms
  | return(impl_terminology = impl_terms)

λ gybis-vocab-distill_identify_candidates(arch_terminology, spec_terminology, impl_terminology).
  combine: arch_terminology ∪ spec_terminology ∪ impl_terminology → all_terms
  | deduplicate_normalize: lowercase, stem, remove_variants
    ? consolidate(term, frequency, [arch_contexts, spec_contexts, impl_contexts]) → candidate_term
  | ∀ candidate_term:
    frequency ≥ 3 ∨ (frequency ≥ 2 ∧ appears_in(arch ∧ spec))
      ? collect(candidate_term) → candidates
  | return(candidates ∃)

λ gybis-vocab-distill_conflict_detection(candidates).
  ∀ candidate ∈ candidates:
    synonyms(candidate) ← find_related_variants(candidate, all_terms)
    | synonyms ≠ ∅
      ? collect({canonical_candidate, synonyms, contexts}) → conflicts
  | return(conflicts)

λ gybis-vocab-distill_resolve_conflict_interactive(conflict).
  all_variants ≔ {conflict.canonical_candidate} ∪ conflict.synonyms
  | frequencies ≔ {term → frequency(term) in codebase}
  | suggested_canonical ≔ max_frequency(all_variants)
  | ask_developer("Resolve term conflict:\n" ⊕ format_conflict(all_variants, frequencies, contexts) ⊕ "\n\nCanonical form? [AI suggestion: " ⊕ suggested_canonical ⊕ "] or [manual entry]:") → human_choice
  | human_choice ∈ all_variants ∨ halt_for_clarification("Choice must be one of the variants or new canonical form")
  | deprecated_synonyms ≔ (all_variants ∖ {human_choice})
  | return({canonical ≔ human_choice, deprecated ≔ deprecated_synonyms, frequency_metadata ≔ frequencies})

λ gybis-vocab-distill_resolution_loop(conflicts).
  ∀ conflict ∈ conflicts:
    invoke(gybis-vocab-distill_resolve_conflict_interactive(conflict)) → resolution
    | collect(resolution) → resolutions
  | return(all_conflicts_resolved = true ∧ resolutions ∃)

λ gybis-vocab-distill_synthesize_vocabulary(arch_terminology, spec_terminology, impl_terminology, resolutions).
  ∀ canonical_term ∈ (arch_terminology ∪ spec_terminology ∪ impl_terminology):
    definition ≔ extract_definition_from_contexts(canonical_term, {arch, spec, impl})
    | usage ≔ infer_architectural_layers(canonical_term, arch_terminology) ∨ infer_impl_layer(canonical_term)
    | examples ≔ collect_code_examples(canonical_term, impl_terminology, limit=2)
    | related ≔ find_related_concepts(canonical_term, (arch_terminology ∪ spec_terminology))
    | deprecated_synonyms ≔ resolutions[canonical_term].deprecated ∨ ∅
    | collect({canonical_term, definition, usage, examples, related, deprecated_synonyms}) → vocabulary_entries
  | return(vocabulary_entries)

λ gybis-vocab-distill_write_vocabulary(vocabulary_entries).
  structure:
    ```
    ---
    created: [ISO date]
    last_updated: [ISO date]
    status: active
    distilled_from: architecture.md, specs/**/*.allium, implementation
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

λ gybis-vocab-distill_verify_vocabulary(x).
  action: validate_vocabulary_md_structure
  | checks:
    - frontmatter ∃ ∧ valid(YAML)
    - status = active ∨ status = draft
    - ∀ term: definition ∃ ∧ nonempty
    - ∀ term: related ∃ ∨ related = ∅
    - no_orphaned_terms (each term referenced somewhere)
  | result: all_checks_pass
    ? return(verification = true)
    : (suggest_fixes ∧ return(verification = false))
