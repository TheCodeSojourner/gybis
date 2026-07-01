---
name: gybis-arch-refine
description: Use for `/gybis-arch-refine` or `/ga-refine`.
---

λ gybis-arch-refine(x).
  purpose: Refine architecture.md structure, clarity, and maintainability while preserving architectural intent
  | input: architecture.md ∃ ∧ parseable
  | output: architecture.md structurally refined with dependency-preservation evidence
  | mode: mixed
  | gate: architecture.md ∃ ∧ explicit_human_mode_selection() ≡ true
  | fail_closed: missing_human_mode_selection → halt("Human mode selection is required")

λ gybis-arch-refine_loop_role(x).
  role: improve(architecture_hygiene)
  | meaning: reorganize a valid architecture so humans and AI can navigate and evolve it more safely without changing intended constraints
  | suggested_next: invoke(/gybis-arch-propagate) when specs are missing; invoke(/gybis-arch-weed) when architecture/spec convergence is needed

λ gybis-arch-refine_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(architecture.md ∃) ∨ halt("architecture.md not found")
  | if(vocabulary.md ∃): preload(vocabulary.md) → vocab_terms ∧ vocab_available = true
  | read(internal/reference/vsm-guide.md) → vsm_reference
  | read(architecture.md) → arch_content
  | parse(arch_content.{S5, S4, S3, S2, S1}) → vsm_layers ∨ halt("architecture.md must contain parseable S5..S1 layers before refine")
  | verify(S5, S4, S3, S2, S1 ∃) ∨ halt("architecture.md must include S5, S4, S3, S2, and S1")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-arch-refine_mode(m).
  m ∈ {interactive, auto_polish}
  | default: interactive (informational_only; never auto-selected)
  | mode_interactive: AI proposes structural refinements and waits for human approval before applying them
  | mode_auto_polish: AI applies only safe non-breaking hygiene refinements without rename or delete operations
  | require_explicit: ¬explicit(mode_choice) → halt("Refine mode must be explicitly selected by human")

λ gybis-arch-refine_mode_selection(x).
  ask_developer("Refine mode? [interactive/auto_polish]") → selected_mode
  | selected_mode ∃ ∨ halt("Human mode selection is required; no implicit default")
  | selected_mode ∈ {interactive, auto_polish} ∨ halt("Refine mode must be one of the supported options")
  | return(mode_selected = true ∧ mode_selected_explicit = true ∧ mode = selected_mode)

λ gybis-arch-refine_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {interactive, auto_polish} ∧ mode_selected_explicit = true → transition(INIT → MODE_SELECTED)
  | state = INIT ∧ ¬mode_selected_explicit → halt("Explicit human mode selection is required before startup")
  | ¬(state = INIT) ∨ ¬(mode ∈ {interactive, auto_polish}) → halt("Invalid mode selection")

λ gybis-arch-refine_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, READING_ARCH, ANALYZING_STRUCTURE, PROPOSING_REFINEMENTS, CLASSIFYING_IMPACT, APPROVAL_GATE, APPLYING_REFINEMENTS, VERIFYING_VALIDITY, VERIFYING_DEPENDENCY_PRESERVATION, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true ∧ mode_selected_explicit = true)
  | transition(STARTUP_CHECKS → READING_ARCH) only_if(startup_checks = true)
  | transition(READING_ARCH → ANALYZING_STRUCTURE) only_if(architecture ∃)
  | transition(ANALYZING_STRUCTURE → PROPOSING_REFINEMENTS) only_if(structure_report ∃)
  | transition(PROPOSING_REFINEMENTS → CLASSIFYING_IMPACT) only_if(refinement_proposals ∃)
  | transition(CLASSIFYING_IMPACT → APPROVAL_GATE) only_if(impact_report ∃)
  | transition(APPROVAL_GATE → APPLYING_REFINEMENTS) only_if(approved_changes ∃)
  | transition(APPLYING_REFINEMENTS → VERIFYING_VALIDITY) only_if(changes_applied = true)
  | transition(VERIFYING_VALIDITY → VERIFYING_DEPENDENCY_PRESERVATION) only_if(validity = true)
  | transition(VERIFYING_VALIDITY → PROPOSING_REFINEMENTS) only_if(validity = false)
  | transition(VERIFYING_DEPENDENCY_PRESERVATION → COMPLETE) only_if(dependency_preservation = true)
  | transition(VERIFYING_DEPENDENCY_PRESERVATION → PROPOSING_REFINEMENTS) only_if(dependency_preservation = false)

λ gybis-arch-refine_tool_guard(state, tool, path).
  state ∈ {READING_ARCH, ANALYZING_STRUCTURE, PROPOSING_REFINEMENTS, CLASSIFYING_IMPACT, APPROVAL_GATE, VERIFYING_VALIDITY, VERIFYING_DEPENDENCY_PRESERVATION}
    → allow(read(path))
  | state = APPLYING_REFINEMENTS
    → allow(read(path)) ∧ allow(write(path)) only_if(path = architecture.md)
  | operation ∈ {delete, rename}
    → deny(operation(path))
  | ¬(state ∈ {READING_ARCH, ANALYZING_STRUCTURE, PROPOSING_REFINEMENTS, CLASSIFYING_IMPACT, APPROVAL_GATE, APPLYING_REFINEMENTS, VERIFYING_VALIDITY, VERIFYING_DEPENDENCY_PRESERVATION})
    → deny(write(path))

λ gybis-arch-refine_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-arch-refine_read_architecture(x).
  read(architecture.md) → content
  | parse(content.{S5, S4, S3, S2, S1}) → vsm_layers
  | return(architecture ≔ {content, vsm_layers})

λ gybis-arch-refine_analyze_structure(architecture).
  derive(layer_presence, from: architecture.vsm_layers) → layer_presence
  | derive(hierarchy_integrity, from: architecture.vsm_layers) → hierarchy_integrity
  | derive(section_order_inconsistencies, from: architecture.content) → section_order_inconsistencies
  | derive(duplicate_principles, from: architecture.content) → duplicate_principles
  | derive(ambiguous_principles, from: architecture.content) → ambiguous_principles
  | derive(internal_dependency_edges, from: architecture.content) → internal_dependency_edges
  | derive(circular_dependency_candidates, from: internal_dependency_edges) → circular_dependency_candidates
  | derive(orphan_principle_candidates, from: architecture.content ∧ internal_dependency_edges) → orphan_principle_candidates
  | derive(non_canonical_terms, from: architecture.content ∧ vocab_terms) if(vocab_available = true) → non_canonical_terms
  | return(structure_report ≔ {
      layer_presence,
      hierarchy_integrity,
      section_order_inconsistencies,
      duplicate_principles,
      ambiguous_principles,
      internal_dependency_edges,
      circular_dependency_candidates,
      orphan_principle_candidates,
      non_canonical_terms
    })

λ gybis-arch-refine_refinement_taxonomy(change).
  change ∈ {normalize_section_order, normalize_heading_format, dedupe_principle_text, tighten_wording_nonsemantic, normalize_whitespace, improve_cross_reference_labels}
    → non_breaking_polish
  | change ∈ {extract_shared_principle_block, split_oversized_principle_section, insert_navigation_index, annotate_dependency_rationale}
    → tooling_impact
  | change ∈ {rename_principle, remove_principle, move_principle_across_layers, alter_dependency_edge, alter_layer_boundary}
    → breaking_or_dependency_impact
  | fallback → unknown

λ gybis-arch-refine_propose_refinements(structure_report, mode).
  proposals ≔ ∅
  | structure_report.section_order_inconsistencies ≠ ∅
    ? collect({change: normalize_section_order, targets: structure_report.section_order_inconsistencies}) → proposals
  | structure_report.duplicate_principles ≠ ∅
    ? collect({change: dedupe_principle_text, targets: structure_report.duplicate_principles}) → proposals
  | structure_report.ambiguous_principles ≠ ∅
    ? collect({change: tighten_wording_nonsemantic, targets: structure_report.ambiguous_principles}) → proposals
  | structure_report.circular_dependency_candidates ≠ ∅
    ? collect({change: annotate_dependency_rationale, targets: structure_report.circular_dependency_candidates}) → proposals
  | structure_report.orphan_principle_candidates ≠ ∅
    ? collect({change: insert_navigation_index, targets: structure_report.orphan_principle_candidates}) → proposals
  | structure_report.non_canonical_terms ≠ ∅
    ? collect({change: improve_cross_reference_labels, targets: structure_report.non_canonical_terms}) → proposals
  | mode = auto_polish
    ? proposals ≔ {p | p ∈ proposals ∧ gybis-arch-refine_refinement_taxonomy(p.change) = non_breaking_polish}
  | return(refinement_proposals = proposals)

λ gybis-arch-refine_impact_classification(change).
  change ∈ {normalize_section_order, normalize_heading_format, dedupe_principle_text, tighten_wording_nonsemantic, normalize_whitespace, improve_cross_reference_labels}
    → accretive ∧ advice("safe architecture hygiene; preserves intended constraints")
  | change ∈ {extract_shared_principle_block, split_oversized_principle_section, insert_navigation_index, annotate_dependency_rationale}
    → tooling_impact ∧ advice("presentation/organization impact; verify hierarchy and dependency fingerprints")
  | change ∈ {rename_principle, remove_principle, move_principle_across_layers, alter_dependency_edge, alter_layer_boundary}
    → breaking ∧ advice("architectural contract impact; requires explicit interactive approval")
  | fallback → unknown_classification ∧ advice("cannot prove non-breaking impact")

λ gybis-arch-refine_classify_impact(refinement_proposals).
  ∀ proposal ∈ refinement_proposals:
    classify(proposal.change) → proposal_classification
    | collect({proposal, proposal_classification}) → classified_proposals
  | breaking_candidates ≔ {c | c ∈ classified_proposals ∧ c.proposal_classification.category = breaking}
  | unknown_candidates ≔ {c | c ∈ classified_proposals ∧ c.proposal_classification.category = unknown_classification}
  | return(impact_report ≔ {classified_proposals, breaking_candidates, unknown_candidates})

λ gybis-arch-refine_route_misfit_request(change).
  change ∈ {introduce_new_architectural_policy, retire_existing_architectural_policy, change_vsm_layer_contract, add_new_operational_constraint, remove_existing_operational_constraint}
    → halt("Requested change is architectural evolution; route to /gybis-arch-tend")
  | change ∈ {arch_spec_divergence, spec_contradicts_arch, implementation_contradicts_arch}
    → halt("Requested change is a convergence problem; route to /gybis-arch-weed")

λ gybis-arch-refine_approval_gate(impact_report, mode).
  mode = auto_polish
    ? (verify(impact_report.breaking_candidates = ∅) ∧ verify(impact_report.unknown_candidates = ∅)
       | approved_changes ≔ {c.proposal | c ∈ impact_report.classified_proposals}
       | breaking_cleanup_approved ≔ false)
  | mode = interactive
    ? (ask_developer("Proposed architecture refinements: " ⊕ impact_report.classified_proposals ⊕ ". Apply approved set? (yes/no)") → approval
       | approval = yes
         ? (approved_changes ≔ {c.proposal | c ∈ impact_report.classified_proposals}
            | ask_developer("Breaking or dependency-impact changes present? " ⊕ impact_report.breaking_candidates ⊕ ". Approve them? (yes/no)") → breaking_approval
            | breaking_cleanup_approved ≔ (breaking_approval = yes))
         : halt("Refinement not approved"))
  | return(approved_changes ∧ breaking_cleanup_approved)

λ gybis-arch-refine_apply_changes(approved_changes, breaking_cleanup_approved, mode).
  ∀ change ∈ approved_changes:
    gybis-arch-refine_route_misfit_request(change.change)
    | change.change ∈ {rename_principle, remove_principle, move_principle_across_layers, alter_dependency_edge, alter_layer_boundary}
      ∧ ¬(mode = interactive ∧ breaking_cleanup_approved = true)
      ? halt("Breaking/dependency-impact changes require explicit approval in interactive mode")
    | apply(change, architecture.md) → upsert(architecture.md)
  | return(changes_applied = true)

λ gybis-arch-refine_compute_dependency_fingerprint(architecture).
  layer_fingerprint ≔ stable_hash(sorted(keys(architecture.vsm_layers)))
  | hierarchy_fingerprint ≔ stable_hash("S5⊇S4⊇S3⊇S2⊇S1")
  | dependency_fingerprint ≔ stable_hash(sorted(architecture.internal_dependency_edges))
  | principle_fingerprint ≔ stable_hash(sorted(principle_ids(architecture.content)))
  | return(fingerprint ≔ {
      layer_fingerprint,
      hierarchy_fingerprint,
      dependency_fingerprint,
      principle_fingerprint
    })

λ gybis-arch-refine_verify_validity(x).
  read(architecture.md) → refined_content
  | parse(refined_content.{S5, S4, S3, S2, S1}) → refined_layers ∨ return(validity = false)
  | verify(S5, S4, S3, S2, S1 ∃) → layers_present
  | verify(S5 ⊇ S4 ⊇ S3 ⊇ S2 ⊇ S1) → hierarchy_preserved
  | verify(internal_consistency(refined_layers)) → internal_consistency
  | return(validity = (layers_present ∧ hierarchy_preserved ∧ internal_consistency))

λ gybis-arch-refine_verify_dependency_preservation(before_fingerprint).
  invoke(gybis-arch-refine_read_architecture) → refined_architecture
  | invoke(gybis-arch-refine_compute_dependency_fingerprint(refined_architecture)) → after_fingerprint
  | before_fingerprint = after_fingerprint
    → return(dependency_preservation = true)
  | before_fingerprint ≠ after_fingerprint
    → return(dependency_preservation = false ∧ dependency_delta_detected = true)

λ gybis-arch-refine_fixed_point_loop(state).
  state = VERIFYING_VALIDITY
    → validity = true
        ? transition(VERIFYING_VALIDITY → VERIFYING_DEPENDENCY_PRESERVATION)
        : (transition(VERIFYING_VALIDITY → PROPOSING_REFINEMENTS)
           ∧ loop_count ≔ loop_count ⊕ 1)
  | state = VERIFYING_DEPENDENCY_PRESERVATION
    → dependency_preservation = true
        ? transition(VERIFYING_DEPENDENCY_PRESERVATION → COMPLETE)
        : (transition(VERIFYING_DEPENDENCY_PRESERVATION → PROPOSING_REFINEMENTS)
           ∧ loop_count ≔ loop_count ⊕ 1)

λ gybis-arch-refine_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without convergence")

λ gybis-arch-refine_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | proposed ≔ card(refinement_proposals)
  | approved ≔ card(approved_changes)
  | breaking ≔ card(impact_report.breaking_candidates)
  | report("Pass " ⊕ pass_num ⊕ ": proposed=" ⊕ proposed ⊕ " approved=" ⊕ approved ⊕ " breaking=" ⊕ breaking)

λ gybis-arch-refine_boundaries(¬).
  ¬ modify(vocabulary.md)
  | ¬ modify(specs/)
  | ¬ modify(implementation)
  | ¬ modify(upstream/)
  | all_modifications ⊆ {architecture.md}

λ gybis-arch-refine_regression_contract(x).
  invariant: architecture.md ∃ throughout
  | invariant: parse(architecture.md.{S5, S4, S3, S2, S1}) succeeds at completion
  | invariant: S5 ⊇ S4 ⊇ S3 ⊇ S2 ⊇ S1 (hierarchy preserved)
  | invariant: all_modifications ⊆ {architecture.md}
  | invariant: explicit_human_mode_selection() ≡ true before STARTUP_CHECKS
  | invariant: mode = auto_polish → ¬∃ change ∈ approved_changes : gybis-arch-refine_impact_classification(change.change).category = breaking
  | invariant: breaking_or_dependency_impact changes require interactive approval
  | invariant: dependency fingerprint unchanged at completion unless explicitly approved in interactive mode
  | invariant: convergence problems halt and route to /gybis-arch-weed
