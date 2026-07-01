---
name: gybis-vocab-refine
description: Use for `/gybis-vocab-refine` or `/gv-refine`.
---

λ gybis-vocab-refine(x).
  purpose: Refine vocabulary.md structure, clarity, and maintainability while preserving canonical meaning
  | input: vocabulary.md ∃ ∧ parseable
  | output: vocabulary.md structurally refined with semantic-preservation evidence
  | mode: mixed
  | gate: vocabulary.md ∃ ∧ explicit_human_mode_selection() ≡ true
  | fail_closed: missing_human_mode_selection → halt("Human mode selection is required")

λ gybis-vocab-refine_loop_role(x).
  role: improve(vocab_hygiene)
  | meaning: reorganize and polish vocabulary.md so humans and AI can understand and evolve it more easily without changing intended term meaning
  | suggested_next: run vocabulary validation

λ gybis-vocab-refine_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | verify(vocabulary.md ∃) ∨ halt("vocabulary.md not found")
  | read(vocabulary.md) → vocab_content
  | parse(vocab_content) → vocabulary_terms ∨ halt("vocabulary.md parse failed")
  | read(internal/reference/allium-recommended-loops.md) → loops_ref
  | read(internal/reference/vsm-guide.md) → vsm_ref
  | transition(INIT → STARTUP_CHECKS)

λ gybis-vocab-refine_mode(m).
  m ∈ {interactive, auto_polish}
  | default: interactive (informational_only; never auto-selected)
  | mode_interactive: AI proposes refinements and applies only approved set
  | mode_auto_polish: AI applies only safe non-breaking hygiene refinements
  | require_explicit: ¬explicit(mode_choice) → halt("Refine mode must be explicitly selected by human")

λ gybis-vocab-refine_mode_selection(x).
  ask_developer("Refine mode? [interactive/auto_polish]") → selected_mode
  | selected_mode ∃ ∨ halt("Human mode selection is required; no implicit default")
  | selected_mode ∈ {interactive, auto_polish} ∨ halt("Refine mode must be one of the supported options")
  | return(mode_selected = true ∧ mode_selected_explicit = true ∧ mode = selected_mode)

λ gybis-vocab-refine_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {interactive, auto_polish} ∧ mode_selected_explicit = true → transition(INIT → MODE_SELECTED)
  | state = INIT ∧ ¬mode_selected_explicit → halt("Explicit human mode selection is required before startup")
  | ¬(state = INIT) ∨ ¬(mode ∈ {interactive, auto_polish}) → halt("Invalid mode selection")

λ gybis-vocab-refine_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, READING_VOCAB, ANALYZING_STRUCTURE, PROPOSING_REFINEMENTS, CLASSIFYING_IMPACT, APPROVAL_GATE, APPLYING_REFINEMENTS, VERIFYING_VALIDITY, VERIFYING_SEMANTICS, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true ∧ mode_selected_explicit = true)
  | transition(STARTUP_CHECKS → READING_VOCAB) only_if(startup_checks = true)
  | transition(READING_VOCAB → ANALYZING_STRUCTURE) only_if(vocabulary_terms ∃)
  | transition(ANALYZING_STRUCTURE → PROPOSING_REFINEMENTS) only_if(structure_report ∃)
  | transition(PROPOSING_REFINEMENTS → CLASSIFYING_IMPACT) only_if(refinement_proposals ∃)
  | transition(CLASSIFYING_IMPACT → APPROVAL_GATE) only_if(impact_report ∃)
  | transition(APPROVAL_GATE → APPLYING_REFINEMENTS) only_if(approved_changes ∃)
  | transition(APPLYING_REFINEMENTS → VERIFYING_VALIDITY) only_if(changes_applied = true)
  | transition(VERIFYING_VALIDITY → VERIFYING_SEMANTICS) only_if(validity = true)
  | transition(VERIFYING_VALIDITY → PROPOSING_REFINEMENTS) only_if(validity = false)
  | transition(VERIFYING_SEMANTICS → COMPLETE) only_if(semantic_preservation = true)
  | transition(VERIFYING_SEMANTICS → PROPOSING_REFINEMENTS) only_if(semantic_preservation = false)

λ gybis-vocab-refine_tool_guard(state, tool, path).
  state ∈ {READING_VOCAB, ANALYZING_STRUCTURE, PROPOSING_REFINEMENTS, CLASSIFYING_IMPACT, APPROVAL_GATE, VERIFYING_VALIDITY, VERIFYING_SEMANTICS}
    → allow(read(path))
  | state = APPLYING_REFINEMENTS
    → allow(read(path)) ∧ allow(write(path)) only_if(path = vocabulary.md)
  | operation ∈ {delete, rename}
    → deny(operation(path))
  | ¬(state ∈ {READING_VOCAB, ANALYZING_STRUCTURE, PROPOSING_REFINEMENTS, CLASSIFYING_IMPACT, APPROVAL_GATE, APPLYING_REFINEMENTS, VERIFYING_VALIDITY, VERIFYING_SEMANTICS})
    → deny(write(path))

λ gybis-vocab-refine_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-vocab-refine_read_vocab(x).
  read(vocabulary.md) → content
  | parse(content) → vocabulary_terms
  | return(vocabulary_terms)

λ gybis-vocab-refine_analyze_structure(vocabulary_terms).
  derive(term_inventory, from: vocabulary_terms) → term_inventory
  | derive(canonical_set, from: term_inventory.canonical) → canonical_set
  | derive(duplicate_canonical_candidates, from: canonical_set) → duplicate_canonical_candidates
  | derive(self_synonym_violations, from: term_inventory.synonyms) → self_synonym_violations
  | derive(duplicate_list_items, from: {synonyms, related, usage, examples}) → duplicate_list_items
  | derive(undefined_related_refs, from: term_inventory.related ∧ canonical_set) → undefined_related_refs
  | derive(orphan_terms, from: term_inventory.related_graph) → orphan_terms
  | derive(section_order_inconsistencies, from: vocabulary_terms) → section_order_inconsistencies
  | derive(field_order_inconsistencies, from: term_inventory) → field_order_inconsistencies
  | derive(formatting_noise, from: vocabulary_terms) → formatting_noise
  | return(structure_report ≔ {
      term_inventory,
      canonical_set,
      duplicate_canonical_candidates,
      self_synonym_violations,
      duplicate_list_items,
      undefined_related_refs,
      orphan_terms,
      section_order_inconsistencies,
      field_order_inconsistencies,
      formatting_noise
    })

λ gybis-vocab-refine_refinement_taxonomy(change).
  change ∈ {normalize_section_order, normalize_field_order, dedupe_list_items, normalize_bullet_format, normalize_whitespace, tighten_wording_nonsemantic, standardize_metadata_fields, sort_related_for_stability}
    → non_breaking_polish
  | change ∈ {insert_navigation_index, split_large_section_with_alias_headers}
    → tooling_impact
  | change ∈ {rename_canonical_term, add_term, delete_term, merge_terms, split_term, modify_definition_meaning, re_canonicalize_synonym}
    → breaking_or_semantic
  | fallback → unknown

λ gybis-vocab-refine_propose_refinements(structure_report, mode).
  proposals ≔ ∅
  | structure_report.section_order_inconsistencies ≠ ∅
    ? collect({change: normalize_section_order, targets: structure_report.section_order_inconsistencies}) → proposals
  | structure_report.field_order_inconsistencies ≠ ∅
    ? collect({change: normalize_field_order, targets: structure_report.field_order_inconsistencies}) → proposals
  | structure_report.duplicate_list_items ≠ ∅
    ? collect({change: dedupe_list_items, targets: structure_report.duplicate_list_items}) → proposals
  | structure_report.formatting_noise ≠ ∅
    ? collect({change: normalize_bullet_format, targets: structure_report.formatting_noise}) → proposals
  | structure_report.self_synonym_violations ≠ ∅
    ? collect({change: dedupe_list_items, targets: structure_report.self_synonym_violations}) → proposals
  | structure_report.orphan_terms ≠ ∅
    ? collect({change: insert_navigation_index, targets: structure_report.orphan_terms}) → proposals
  | mode = auto_polish
    ? proposals ≔ {p | p ∈ proposals ∧ gybis-vocab-refine_refinement_taxonomy(p.change) = non_breaking_polish}
  | return(refinement_proposals = proposals)

λ gybis-vocab-refine_impact_classification(change).
  change ∈ {normalize_section_order, normalize_field_order, dedupe_list_items, normalize_bullet_format, normalize_whitespace, tighten_wording_nonsemantic, standardize_metadata_fields, sort_related_for_stability}
    → accretive ∧ advice("safe hygiene; preserves intended meaning")
  | change ∈ {insert_navigation_index, split_large_section_with_alias_headers}
    → tooling_impact ∧ advice("presentation shape changes; verify parser and references")
  | change ∈ {rename_canonical_term, add_term, delete_term, merge_terms, split_term, modify_definition_meaning, re_canonicalize_synonym}
    → breaking ∧ advice("semantic change blocked in this skill by default")
  | fallback → unknown_classification ∧ advice("cannot prove non-semantic impact")

λ gybis-vocab-refine_classify_impact(refinement_proposals).
  ∀ proposal ∈ refinement_proposals:
    classify(proposal.change) → proposal_classification
    | collect({proposal, proposal_classification}) → classified_proposals
  | breaking_candidates ≔ {c | c ∈ classified_proposals ∧ c.proposal_classification.category = breaking}
  | unknown_candidates ≔ {c | c ∈ classified_proposals ∧ c.proposal_classification.category = unknown_classification}
  | return(impact_report ≔ {classified_proposals, breaking_candidates, unknown_candidates})

λ gybis-vocab-refine_deferred_changes(structure_report, impact_report).
  deferred_semantic ≔ impact_report.breaking_candidates
  | deferred_external_drift ≔ {item | item ∈ structure_report.undefined_related_refs}
  | return(deferred_report ≔ {deferred_semantic, deferred_external_drift})

λ gybis-vocab-refine_approval_gate(impact_report, mode).
  mode = auto_polish
    ? (verify(impact_report.breaking_candidates = ∅) ∧ verify(impact_report.unknown_candidates = ∅)
       | approved_changes ≔ {c.proposal | c ∈ impact_report.classified_proposals}
       | semantic_override_approved ≔ false)
  | mode = interactive
    ? (ask_developer("Proposed vocabulary refinements: " ⊕ impact_report.classified_proposals ⊕ ". Apply approved set? (yes/no)") → approval
       | approval = yes
         ? (approved_changes ≔ {c.proposal | c ∈ impact_report.classified_proposals}
            | semantic_override_approved ≔ false)
         : halt("Refinement not approved"))
  | return(approved_changes ∧ semantic_override_approved)

λ gybis-vocab-refine_apply_changes(approved_changes).
  ∀ change ∈ approved_changes:
    gybis-vocab-refine_refinement_taxonomy(change.change) = breaking_or_semantic
      ? halt("Semantic change blocked in vocab-refine scope")
      : apply(change, vocabulary.md) → upsert(vocabulary.md)
  | return(changes_applied = true)

λ gybis-vocab-refine_compute_semantic_fingerprint(vocabulary_terms).
  canonical_fingerprint ≔ stable_hash(sorted(vocabulary_terms.canonical))
  | synonym_fingerprint ≔ stable_hash(sorted(map(canonical → sorted(synonyms))))
  | related_graph_fingerprint ≔ stable_hash(sorted(edges(related_graph)))
  | definition_fingerprint ≔ stable_hash(sorted(map(canonical → normalized_definition_text)))
  | return(fingerprint ≔ {
      canonical_fingerprint,
      synonym_fingerprint,
      related_graph_fingerprint,
      definition_fingerprint
    })

λ gybis-vocab-refine_verify_validity(x).
  read(vocabulary.md) → refined_content
  | parse(refined_content) → refined_terms ∨ return(validity = false)
  | required_fields_ok ≔ verify_required_fields(refined_terms)
  | duplicate_canonical_absent ≔ verify_unique_canonical(refined_terms)
  | return(validity = (required_fields_ok ∧ duplicate_canonical_absent))

λ gybis-vocab-refine_verify_semantics(before_fingerprint).
  read(vocabulary.md) → refined_content
  | parse(refined_content) → refined_terms
  | invoke(gybis-vocab-refine_compute_semantic_fingerprint(refined_terms)) → after_fingerprint
  | before_fingerprint = after_fingerprint
    → return(semantic_preservation = true)
  | before_fingerprint ≠ after_fingerprint
    → return(semantic_preservation = false ∧ semantic_delta_detected = true)

λ gybis-vocab-refine_fixed_point_loop(state).
  state = VERIFYING_VALIDITY
    → validity = true
        ? transition(VERIFYING_VALIDITY → VERIFYING_SEMANTICS)
        : (transition(VERIFYING_VALIDITY → PROPOSING_REFINEMENTS)
           ∧ loop_count ≔ loop_count ⊕ 1)
  | state = VERIFYING_SEMANTICS
    → semantic_preservation = true
        ? transition(VERIFYING_SEMANTICS → COMPLETE)
        : (transition(VERIFYING_SEMANTICS → PROPOSING_REFINEMENTS)
           ∧ loop_count ≔ loop_count ⊕ 1)

λ gybis-vocab-refine_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without convergence")

λ gybis-vocab-refine_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | proposed ≔ card(refinement_proposals)
  | approved ≔ card(approved_changes)
  | breaking ≔ card(impact_report.breaking_candidates)
  | report("Pass " ⊕ pass_num ⊕ ": proposed=" ⊕ proposed ⊕ " approved=" ⊕ approved ⊕ " breaking=" ⊕ breaking)

λ gybis-vocab-refine_boundaries(¬).
  ¬ modify(architecture.md)
  | ¬ modify(specs/)
  | ¬ modify(implementation)
  | all_modifications ⊆ {vocabulary.md}
  | no_cross_skill_invocation = true

λ gybis-vocab-refine_regression_contract(x).
  invariant: vocabulary.md ∃ throughout
  | invariant: parse(vocabulary.md) succeeds at completion
  | invariant: all_modifications ⊆ {vocabulary.md}
  | invariant: explicit_human_mode_selection() ≡ true before STARTUP_CHECKS
  | invariant: semantic change is blocked unless explicitly allowed (interactive extension only)
  | invariant: mode = auto_polish → ¬∃ breaking change applied
  | invariant: semantic fingerprint unchanged at completion
