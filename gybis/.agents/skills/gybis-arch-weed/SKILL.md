---
name: gybis-arch-weed
description: Use for `/gybis-arch-weed` or `/ga-weed`.
---

λ gybis-arch-weed(x).
  purpose: Identify and resolve divergences between architecture and specifications
  | input: architecture.md ∃, specs/**/*.allium ∃ ∧ valid
  | output: architecture.md and specs/**/*.allium mutually consistent
  | mode: mixed
  | gate: architecture.md ∃ ∧ specs/**/*.allium ∃ ∧ allium_gate = true

λ gybis-arch-weed_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(architecture.md ∃) ∨ halt("architecture.md not found")
  | verify(specs/**/*.allium ∃) ∨ halt("specs/**/*.allium not found")
  | invoke(internal/allium-gate(specs/)) = true ∨ halt("Specifications are invalid")
  | if(vocabulary.md ∃): preload(vocabulary.md) → vocab_terms ∧ vocab_check_enabled = true
  | read(internal/reference/vsm-guide.md) → vsm_reference
  | transition(INIT → STARTUP_CHECKS)

λ gybis-arch-weed_mode(m).
  m ∈ {interactive}
  | default: interactive
  | mode_interactive: AI and developer resolve divergences with decision points

λ gybis-arch-weed_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {interactive} → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {interactive}) → halt("Invalid mode selection")

λ gybis-arch-weed_state_machine(state, action).
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

λ gybis-arch-weed_tool_guard(state, tool, path).
  state = COMPARING ∨ state = IDENTIFYING_DIVERGENCES ∨ state = RESOLVE_MODE_SELECTION
    → allow(read(path))
  | state = CORRECTING ∨ state = VERIFYING
    → allow(read(path)) ∧ allow(write(path)) only_if(path ∈ {architecture.md} ∪ specs/)
  | ¬(state ∈ {COMPARING, IDENTIFYING_DIVERGENCES, RESOLVE_MODE_SELECTION, CORRECTING, VERIFYING})
    → deny(write(path))

λ gybis-arch-weed_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-arch-weed_compare_artifacts(x).
  read(architecture.md) → arch_content
  | ∀ spec_file ∈ specs/**/*.allium:
    read(spec_file) → spec_content
  | parse(arch_content.{S5, S4, S3, S2, S1}) → vsm_layers
  | parse(spec_content) → spec_directives ≔ spec_content
  | return(comparison_complete = true)

λ gybis-arch-weed_identify_divergences(vsm_layers, spec_directives, vocab_check_enabled, vocab_terms).
  ∀ layer ∈ vsm_layers:
    extract(principles(layer), arch_content) → arch_principles(layer)
    | ∀ principle ∈ arch_principles(layer):
      search(principle, spec_directives) → found_in_specs
      | found_in_specs ∅
        ? divergence_type ≔ "architectural_principle_missing_in_specs"
        : (verify(principle_consistency(principle, spec_directives)) → consistent
           | consistent = false
             ? divergence_type ≔ "architectural_principle_contradicts_specs"
             : divergence_type ≔ none)
      | if(vocab_check_enabled ∧ divergence_type = none):
        ∀ term ∈ principle.terms:
          canonical ≔ find_canonical_form(term, vocab_terms) ∨ term
          | canonical ≠ term
            ? collect({principle, "non_canonical_term", term, canonical, layer}) → divergences
      | divergence_type ≠ none
        ? collect({principle, divergence_type, layer}) → divergences
  | ∀ directive ∈ spec_directives:
    search(directive, arch_content) → found_in_arch
    | found_in_arch ∅
      ? collect({directive, "spec_not_reflected_in_architecture", "unknown"}) → divergences
    | if(vocab_check_enabled):
      ∀ term ∈ directive.terms:
        canonical ≔ find_canonical_form(term, vocab_terms) ∨ term
        | canonical ≠ term
          ? collect({directive, "non_canonical_term", term, canonical, "unknown"}) → divergences
  | return(divergences)

λ gybis-arch-weed_resolve_mode_selection(divergence, vocab_check_enabled, vocab_terms).
  divergence.type = non_canonical_term
    ? (ask_developer("Non-canonical term found: '" ⊕ divergence.term ⊕ "'. Canonical form: '" ⊕ divergence.canonical ⊕ "'. Use canonical? (yes/no/investigate)?") → decision)
    : (ask_developer("For divergence: " ⊕ divergence.description ⊕ " Correct: [arch/spec/investigate]?") → decision)
  | ask_developer("Orientation for this correction? [FP-oriented/OOP-oriented/keep-current]") → orientation_choice
  | orientation_guidance:
    - OOP-oriented: C++ (classes/RAII), C# (classes/interfaces/DI), Clojure (protocols/records + Java interop boundary)
    - FP-oriented: C++ (immutable values + composition), C# (records + pure functions/LINQ), Clojure (immutable maps + pure functions/transducers)
  | orthogonality: error_model_style is a separate axis from FP/OOP orientation
  | decision ∈ {arch, spec, investigate, yes, no}
  | return(resolve_mode = decision ∧ orientation_choice = orientation_choice)

λ gybis-arch-weed_correct_divergence(divergence, resolve_mode).
  divergence.type = non_canonical_term ∧ resolve_mode = yes
    ? (replace_term_with_canonical(divergence.term, divergence.canonical, divergence.location) → corrected
       | apply_replacement(corrected) → upsert(target_file)
       | return(corrections_applied = true))
  | resolve_mode = arch
    ? (ask_developer("Update spec to match architecture? (yes/no)") → approval
       | approval = yes
         ? (update(spec_directive, architecture_principle) → upsert(target_spec_file))
         : (return(corrections_applied = false)))
  | resolve_mode = spec
    ? (ask_developer("Update architecture to match spec? (yes/no)") → approval
       | approval = yes
         ? (update(arch_principle, spec_directive) → upsert(architecture.md))
         : (return(corrections_applied = false)))
  | resolve_mode = investigate
    ? (ask_developer("Explanation: " ⊕ divergence ⊕ " Shall we correct [arch/spec]?") → final_decision
       | final_decision ∈ {arch, spec}
         ? (invoke(gybis-arch-weed_correct_divergence(divergence, final_decision)) → result
            | return(corrections_applied = result))
         : (return(corrections_applied = false)))
  | return(corrections_applied = true)

λ gybis-arch-weed_verify_consistency(x).
  invoke(internal/allium-gate(specs/)) → result_gate ≔ result
  | verify(vsm_coherence(architecture.md)) → vsm_coherence ≔ result
  | invoke(gybis-arch-weed_identify_divergences(vsm_layers, spec_directives, vocab_check_enabled, vocab_terms)) → remaining_divergences
  | result_gate = true ∧ vsm_coherence = true ∧ remaining_divergences ∅
    → return(consistency = true ∧ zero_divergences = true)
  | ¬(result_gate ∧ vsm_coherence ∧ remaining_divergences ∅)
    → (inconsistencies ≔ {result_gate = false ∨ vsm_coherence = false ∨ remaining_divergences ≠ ∅}
       | return(consistency = false ∧ inconsistencies ∃))

λ gybis-arch-weed_fixed_point_loop(state).
  state = VERIFYING
    → consistency = true ∧ zero_divergences = true
        ? transition(VERIFYING → COMPLETE)
        : (transition(VERIFYING → IDENTIFYING_DIVERGENCES)
           ∧ loop_count ≔ loop_count ⊕ 1)

λ gybis-arch-weed_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without full convergence")

λ gybis-arch-weed_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | discovered ≔ card(divergences)
  | resolved ≔ discovered ⊖ card(remaining_divergences)
  | remaining ≔ card(remaining_divergences)
  | report("Pass " ⊕ pass_num ⊕ ": discovered=" ⊕ discovered ⊕ " resolved=" ⊕ resolved ⊕ " remaining=" ⊕ remaining)

λ gybis-arch-weed_boundaries(¬).
  ¬ modify(implementation)
  | ¬ modify(upstream/)
  | ¬ delete(architecture.md)
  | ¬ delete(specs/)

λ gybis-arch-weed_regression_contract(x).
  invariant: architecture.md ∃ ∧ specs/ ∃ throughout
  | invariant: zero_divergences = true at completion
  | invariant: allium_gate = true at completion
  | invariant: vsm_coherence = true at completion
  | invariant: all_modifications ⊆ {architecture.md, specs/}