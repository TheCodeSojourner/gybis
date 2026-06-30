---
name: gybis-vocab-weed
description: Use for `/gybis-vocab-weed` or `/gv-weed`.
---

λ gybis-vocab-weed(x).
  purpose: Identify and resolve vocabulary drift between vocabulary.md and downstream artifacts (architecture, specs, implementation)
  | input: vocabulary.md ∃ ∧ (architecture.md ∃ ∨ specs/**/*.allium ∃ ∨ implementation ∃)
  | output: vocabulary.md and downstream artifacts use canonical terms consistently
  | mode: interactive
  | gate: vocabulary.md ∃

λ gybis-vocab-weed_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(vocabulary.md ∃) ∨ halt("vocabulary.md not found")
  | read(vocabulary.md) → vocab_content
  | parse(vocab_content) → vocabulary_terms
  | if(architecture.md ∃): read(architecture.md) → arch_content
  | if(specs/**/*.allium ∃): read_all_specs → spec_content
  | if(implementation ∃): recursively_read(implementation) → impl_content
  | verify(architecture.md ∃ ∨ specs/**/*.allium ∃ ∨ implementation ∃) ∨ halt("No downstream artifacts found (need architecture.md, specs/**/*.allium, or implementation)")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-vocab-weed_mode(m).
  m ∈ {interactive}
  | default: interactive
  | mode_interactive: AI and developer resolve vocabulary drift with explicit decisions

λ gybis-vocab-weed_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {interactive} → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {interactive}) → halt("Invalid mode selection")

λ gybis-vocab-weed_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, COMPARING, IDENTIFYING_DIVERGENCES, RESOLVE_MODE_SELECTION, CORRECTING, VERIFYING, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → COMPARING) only_if(startup_checks = true)
  | transition(COMPARING → IDENTIFYING_DIVERGENCES) only_if(comparison_complete = true)
  | transition(IDENTIFYING_DIVERGENCES → RESOLVE_MODE_SELECTION) only_if(divergences ∃)
  | transition(RESOLVE_MODE_SELECTION → CORRECTING) only_if(resolve_mode ∃)
  | transition(CORRECTING → VERIFYING) only_if(corrections_applied = true)
  | transition(VERIFYING → IDENTIFYING_DIVERGENCES) only_if(inconsistencies ∃)
  | transition(VERIFYING → COMPLETE) only_if(consistency = true ∧ zero_divergences = true)

λ gybis-vocab-weed_tool_guard(state, tool, path).
  state = COMPARING ∨ state = IDENTIFYING_DIVERGENCES ∨ state = RESOLVE_MODE_SELECTION
    → allow(read(path))
  | state = CORRECTING ∨ state = VERIFYING
    → allow(read(path)) ∧ allow(write(path)) only_if(path ∈ {vocabulary.md, architecture.md} ∪ specs/ ∪ implementation)
  | ¬(state ∈ {COMPARING, IDENTIFYING_DIVERGENCES, RESOLVE_MODE_SELECTION, CORRECTING, VERIFYING})
    → deny(write(path))

λ gybis-vocab-weed_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-vocab-weed_compare_artifacts(vocabulary_terms, arch_content, spec_content, impl_content).
  divergences ≔ ∅
  | comparison_complete ≔ false
  | canonical_terms ≔ map(term_name, vocabulary_terms)
  | alias_map ≔ map(alias, canonical_term, vocabulary_terms)
  | if(architecture.md ∃):
    ∀ term ∈ extract_terms(arch_content):
      canonical ≔ find_canonical_form(term, vocabulary_terms) ∨ null
      | canonical ≠ null ∧ canonical ≠ term
        ? collect({type: "non_canonical_term_in_arch", term, canonical}) → divergences
      | canonical = null ∧ looks_like_domain_term(term)
        ? collect({type: "undefined_term_in_arch", term}) → divergences
  | if(specs/**/*.allium ∃):
    ∀ term ∈ extract_terms(spec_content):
      canonical ≔ find_canonical_form(term, vocabulary_terms) ∨ null
      | canonical ≠ null ∧ canonical ≠ term
        ? collect({type: "non_canonical_term_in_specs", term, canonical}) → divergences
      | canonical = null ∧ looks_like_domain_term(term)
        ? collect({type: "undefined_term_in_specs", term}) → divergences
  | if(implementation ∃):
    ∀ term ∈ extract_terms(impl_content):
      canonical ≔ find_canonical_form(term, vocabulary_terms) ∨ null
      | canonical ≠ null ∧ canonical ≠ term
        ? collect({type: "non_canonical_term_in_code", term, canonical}) → divergences
      | canonical = null ∧ looks_like_domain_term(term)
        ? collect({type: "undefined_term_in_code", term}) → divergences
  | ∀ canonical ∈ canonical_terms:
    used_in_arch ≔ architecture.md ∃ ∧ search(canonical, arch_content)
    | used_in_specs ≔ specs/**/*.allium ∃ ∧ search(canonical, spec_content)
    | used_in_code ≔ implementation ∃ ∧ search(canonical, impl_content)
    | used_in_arch = false ∧ used_in_specs = false ∧ used_in_code = false
      ? collect({type: "unused_canonical_term", canonical}) → divergences
  | comparison_complete ≔ true
  | return(comparison_complete = comparison_complete ∧ divergences = divergences)

λ gybis-vocab-weed_identify_divergences(comparison_result).
  return(divergences = comparison_result.divergences)

λ gybis-vocab-weed_resolve_mode_selection(divergence).
  divergence.type = "non_canonical_term_in_arch"
    ? (ask_developer("Architecture uses non-canonical term '" ⊕ divergence.term ⊕ "'. Canonical is '" ⊕ divergence.canonical ⊕ "'. Resolve as [arch/vocab/investigate/skip]?") → decision)
  | divergence.type = "non_canonical_term_in_specs"
    ? (ask_developer("Specifications use non-canonical term '" ⊕ divergence.term ⊕ "'. Canonical is '" ⊕ divergence.canonical ⊕ "'. Resolve as [spec/vocab/investigate/skip]?") → decision)
  | divergence.type = "non_canonical_term_in_code"
    ? (ask_developer("Implementation uses non-canonical term '" ⊕ divergence.term ⊕ "'. Canonical is '" ⊕ divergence.canonical ⊕ "'. Resolve as [code/vocab/investigate/skip]?") → decision)
  | divergence.type = "undefined_term_in_arch"
    ? (ask_developer("Architecture uses term '" ⊕ divergence.term ⊕ "' that is undefined in vocabulary.md. Resolve as [arch/vocab/investigate/skip]?") → decision)
  | divergence.type = "undefined_term_in_specs"
    ? (ask_developer("Specifications use term '" ⊕ divergence.term ⊕ "' that is undefined in vocabulary.md. Resolve as [spec/vocab/investigate/skip]?") → decision)
  | divergence.type = "undefined_term_in_code"
    ? (ask_developer("Implementation uses term '" ⊕ divergence.term ⊕ "' that is undefined in vocabulary.md. Resolve as [code/vocab/investigate/skip]?") → decision)
  | divergence.type = "unused_canonical_term"
    ? (ask_developer("Canonical term '" ⊕ divergence.canonical ⊕ "' appears unused across architecture/spec/code. Resolve as [keep/delete/investigate/skip]?") → decision)
  | decision ∈ {arch, spec, code, vocab, keep, delete, investigate, skip}
  | return(resolve_mode = decision)

λ gybis-vocab-weed_correct_divergence(divergence, resolve_mode).
  resolve_mode = arch
    ? (replace_all(divergence.term, divergence.canonical, architecture.md) → updated_arch
       | write(architecture.md) → persisted
       | return(corrections_applied = true))
  | resolve_mode = spec
    ? (replace_all(divergence.term, divergence.canonical, specs/) → updated_specs
       | write(specs/) → persisted
       | return(corrections_applied = true))
  | resolve_mode = code
    ? (replace_all(divergence.term, divergence.canonical, implementation) → updated_impl
       | write(implementation) → persisted
       | return(corrections_applied = true))
  | resolve_mode = vocab
    ? (ask_developer("Update vocabulary.md to include or re-canonicalize this term? (yes/no)") → approval
       | approval = yes
         ? (upsert(vocabulary.md, apply_vocab_change(divergence)) → result
            | return(corrections_applied = true))
         : return(corrections_applied = false))
  | resolve_mode = delete ∧ divergence.type = "unused_canonical_term"
    ? (remove_term(vocabulary.md, divergence.canonical) → updated_vocab
       | write(vocabulary.md) → persisted
       | return(corrections_applied = true))
  | resolve_mode = keep ∧ divergence.type = "unused_canonical_term"
    ? return(corrections_applied = false)
  | resolve_mode = investigate
    ? (ask_developer("Analysis: " ⊕ divergence ⊕ " Choose [arch/spec/code/vocab/keep/delete/skip]") → final_decision
       | final_decision ∈ {arch, spec, code, vocab, keep, delete, skip}
         ? invoke(gybis-vocab-weed_correct_divergence(divergence, final_decision)) → result
         : return(corrections_applied = false))
  | resolve_mode = skip
    ? return(corrections_applied = false)
  | return(corrections_applied = true)

λ gybis-vocab-weed_verify_consistency(x).
  read(vocabulary.md) → vocab_content
  | parse(vocab_content) → vocabulary_terms
  | if(architecture.md ∃): read(architecture.md) → arch_content
  | if(specs/**/*.allium ∃): read_all_specs → spec_content
  | if(implementation ∃): recursively_read(implementation) → impl_content
  | invoke(gybis-vocab-weed_compare_artifacts(vocabulary_terms, arch_content, spec_content, impl_content)) → comparison
  | invoke(gybis-vocab-weed_identify_divergences(comparison)) → remaining_divergences
  | remaining_divergences ∅
    → return(consistency = true ∧ zero_divergences = true)
  | remaining_divergences ≠ ∅
    → return(consistency = false ∧ inconsistencies = remaining_divergences)

λ gybis-vocab-weed_fixed_point_loop(state).
  state = VERIFYING
    → consistency = true ∧ zero_divergences = true
        ? transition(VERIFYING → COMPLETE)
        : (transition(VERIFYING → IDENTIFYING_DIVERGENCES)
           ∧ loop_count ≔ loop_count ⊕ 1)

λ gybis-vocab-weed_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without full convergence")

λ gybis-vocab-weed_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | discovered ≔ card(divergences)
  | resolved ≔ discovered ⊖ card(remaining_divergences)
  | remaining ≔ card(remaining_divergences)
  | report("Pass " ⊕ pass_num ⊕ ": discovered=" ⊕ discovered ⊕ " resolved=" ⊕ resolved ⊕ " remaining=" ⊕ remaining)

λ gybis-vocab-weed_boundaries(¬).
  ¬ modify(upstream/)
  | ¬ delete(vocabulary.md)

λ gybis-vocab-weed_regression_contract(x).
  invariant: vocabulary.md ∃ throughout
  | invariant: zero_divergences = true at completion
  | invariant: all_modifications ⊆ {vocabulary.md, architecture.md, specs/, implementation}
  | invariant: divergence.type ∈ {"non_canonical_term_in_arch", "non_canonical_term_in_specs", "non_canonical_term_in_code", "undefined_term_in_arch", "undefined_term_in_specs", "undefined_term_in_code", "unused_canonical_term"}