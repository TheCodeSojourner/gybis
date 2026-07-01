---
name: gybis-spec-refine
description: Use for `/gybis-spec-refine` or `/gs-refine`.
---

λ gybis-spec-refine(x).
  purpose: Refine valid specifications for structure, clarity, and maintainability while preserving intended behavior
  | input: specs/**/*.allium ∃ ∧ valid
  | output: specs/**/*.allium structurally refined with behavior-preservation evidence
  | mode: mixed
  | gate: specs/**/*.allium ∃ ∧ allium_gate = true

λ gybis-spec-refine_loop_role(x).
  role: improve(spec_hygiene)
  | meaning: reorganize a valid spec tree so humans and AI can understand, navigate, and evolve it more easily without changing intended behavior
  | suggested_next: invoke(/gybis-spec-propagate) → run_tests → invoke(/gybis-spec-weed)

λ gybis-spec-refine_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | preload: [internal/allium-check, internal/allium-analyse, internal/allium-normalize, internal/allium-gate, internal/allium-plan]
  | if(vocabulary.md ∃): preload(vocabulary.md) → vocab_terms ∧ vocab_available = true
  | read(internal/reference/allium-language-reference.md) → language_ref
  | read(internal/reference/allium-patterns.md) → patterns_ref
  | read(internal/reference/allium-recommended-loops.md) → loops_ref
  | read(internal/reference/allium-constructs.md) → constructs_registry
  | read(internal/reference/allium-actioning-findings.md) → findings_ref
  | verify(specs/**/*.allium ∃) ∨ halt("specs/**/*.allium not found")
  | invoke(internal/allium-gate(specs/)) = true ∨ halt("Specifications are invalid; run /gybis-spec-check before /gybis-spec-refine")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-spec-refine_mode(m).
  m ∈ {interactive, auto_polish}
  | default: interactive
  | mode_interactive: AI proposes structural refinements and waits for human approval before applying them
  | mode_auto_polish: AI applies only safe non-breaking hygiene refinements without delete or rename operations

λ gybis-spec-refine_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {interactive, auto_polish} → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {interactive, auto_polish}) → halt("Invalid mode selection")

λ gybis-spec-refine_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, READING_SPECS, ANALYZING_STRUCTURE, PROPOSING_REFINEMENTS, CLASSIFYING_IMPACT, APPROVAL_GATE, APPLYING_REFINEMENTS, VERIFYING_VALIDITY, VERIFYING_OBLIGATIONS, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → READING_SPECS) only_if(startup_checks = true)
  | transition(READING_SPECS → ANALYZING_STRUCTURE) only_if(specifications ∃)
  | transition(ANALYZING_STRUCTURE → PROPOSING_REFINEMENTS) only_if(structure_report ∃)
  | transition(PROPOSING_REFINEMENTS → CLASSIFYING_IMPACT) only_if(refinement_proposals ∃)
  | transition(CLASSIFYING_IMPACT → APPROVAL_GATE) only_if(impact_report ∃)
  | transition(APPROVAL_GATE → APPLYING_REFINEMENTS) only_if(approved_changes ∃)
  | transition(APPLYING_REFINEMENTS → VERIFYING_VALIDITY) only_if(changes_applied = true)
  | transition(VERIFYING_VALIDITY → VERIFYING_OBLIGATIONS) only_if(validity = true)
  | transition(VERIFYING_VALIDITY → PROPOSING_REFINEMENTS) only_if(validity = false)
  | transition(VERIFYING_OBLIGATIONS → COMPLETE) only_if(obligation_preservation = true)
  | transition(VERIFYING_OBLIGATIONS → PROPOSING_REFINEMENTS) only_if(obligation_preservation = false)

λ gybis-spec-refine_tool_guard(state, tool, path).
  state = READING_SPECS ∨ state = ANALYZING_STRUCTURE ∨ state = PROPOSING_REFINEMENTS ∨ state = CLASSIFYING_IMPACT ∨ state = APPROVAL_GATE ∨ state = VERIFYING_VALIDITY ∨ state = VERIFYING_OBLIGATIONS
    → allow(read(path))
  | state = APPLYING_REFINEMENTS
    → allow(read(path)) ∧ allow(write(path)) only_if(path ⊆ specs/)
  | operation ∈ {delete, rename}
    → allow(operation(path)) only_if(state = APPLYING_REFINEMENTS ∧ path ⊆ specs/ ∧ mode = interactive ∧ breaking_cleanup_approved = true)
  | ¬(state ∈ {READING_SPECS, ANALYZING_STRUCTURE, PROPOSING_REFINEMENTS, CLASSIFYING_IMPACT, APPROVAL_GATE, APPLYING_REFINEMENTS, VERIFYING_VALIDITY, VERIFYING_OBLIGATIONS})
    → deny(write(path))

λ gybis-spec-refine_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-spec-refine_read_specs(x).
  ∀ spec_file ∈ specs/**/*.allium:
    read(spec_file) → content
    | collect({path: spec_file, content: content}) → specifications
  | return(specifications)

λ gybis-spec-refine_analyze_structure(specifications).
  derive(domain_file_map, from: specifications.paths) → domain_file_map
  | derive(import_graph, from: specifications.content) → import_graph
  | derive(declaration_inventory, from: specifications.content) → declaration_inventory
  | derive(contract_entity_reference_map, from: specifications.content) → contract_entity_reference_map
  | derive(surface_exposure_map, from: specifications.content) → surface_exposure_map
  | derive(duplicated_policy_fragments, from: specifications.content) → duplicated_policy_fragments
  | derive(compatibility_wrapper_candidates, from: specifications.content) → compatibility_wrapper_candidates
  | derive(root_level_clutter, from: specifications.paths) → root_level_clutter
  | derive(naming_inconsistencies, from: specifications.content) → naming_inconsistencies
  | derive(isolated_modules, from: import_graph ∧ declaration_inventory) → isolated_modules
  | return(structure_report ≔ {
      domain_file_map,
      import_graph,
      declaration_inventory,
      contract_entity_reference_map,
      surface_exposure_map,
      duplicated_policy_fragments,
      compatibility_wrapper_candidates,
      root_level_clutter,
      naming_inconsistencies,
      isolated_modules
    })

λ gybis-spec-refine_refinement_taxonomy(change).
  change ∈ {warning_cleanup, readability_cleanup, naming_alignment, additive_surface_for_local_reference, additive_composition_wrapper}
    → non_breaking_polish
  | change ∈ {split_oversized_module, extract_shared_module, introduce_domain_local_module}
    → structural_split
  | change ∈ {merge_tiny_modules, collapse_redundant_wrapper_layers}
    → structural_merge
  | change ∈ {move_declaration_between_modules, move_file_between_domains}
    → module_move
  | change ∈ {rename_module, rename_file_and_rewrite_imports}
    → rename_with_import_rewrite
  | change ∈ {delete_compatibility_wrapper, delete_transitional_module, delete_dead_module}
    → orphan_cleanup
  | fallback → unknown

λ gybis-spec-refine_propose_refinements(structure_report, mode).
  proposals ≔ ∅
  | structure_report.duplicated_policy_fragments ≠ ∅
    ? collect({change: extract_shared_module, targets: structure_report.duplicated_policy_fragments}) → proposals
  | structure_report.compatibility_wrapper_candidates ≠ ∅
    ? collect({change: delete_compatibility_wrapper, targets: structure_report.compatibility_wrapper_candidates}) → proposals
  | structure_report.root_level_clutter ≠ ∅
    ? collect({change: move_file_between_domains, targets: structure_report.root_level_clutter}) → proposals
  | structure_report.naming_inconsistencies ≠ ∅
    ? collect({change: naming_alignment, targets: structure_report.naming_inconsistencies}) → proposals
  | structure_report.isolated_modules ≠ ∅
    ? collect({change: additive_surface_for_local_reference, targets: structure_report.isolated_modules}) → proposals
  | mode = auto_polish
    ? proposals ≔ {p | p ∈ proposals ∧ gybis-spec-refine_refinement_taxonomy(p.change) ∈ {non_breaking_polish, structural_split}}
  | return(refinement_proposals = proposals)

λ gybis-spec-refine_impact_classification(change).
  change ∈ {warning_cleanup, readability_cleanup, naming_alignment, additive_surface_for_local_reference, additive_composition_wrapper, extract_shared_module, introduce_domain_local_module}
    → accretive ∧ advice("safe to apply without changing intended behavior; still verify with allium-check, allium-analyse, and allium-plan")
  | change ∈ {split_oversized_module, merge_tiny_modules, collapse_redundant_wrapper_layers, move_declaration_between_modules, move_file_between_domains}
    → tooling_impact ∧ advice("meaning preserved, but analyzer surfaces, import structure, or obligation planning shape may change; verify before and after")
  | change ∈ {rename_module, rename_file_and_rewrite_imports, delete_compatibility_wrapper, delete_transitional_module, delete_dead_module}
    → breaking ∧ advice("visible module/file surface changes require explicit human approval before apply")
  | fallback → unknown_classification ∧ advice("cannot prove structural-only impact; route to human review or /gybis-spec-tend")

λ gybis-spec-refine_classify_impact(refinement_proposals).
  ∀ proposal ∈ refinement_proposals:
    classify(proposal.change) → proposal_classification
    | collect({proposal, proposal_classification}) → classified_proposals
  | breaking_candidates ≔ {c | c ∈ classified_proposals ∧ c.proposal_classification.category = breaking}
  | unknown_candidates ≔ {c | c ∈ classified_proposals ∧ c.proposal_classification.category = unknown_classification}
  | return(impact_report ≔ {classified_proposals, breaking_candidates, unknown_candidates})

λ gybis-spec-refine_route_misfit_request(change).
  change ∈ {rename_entity, rename_contract, rename_field, change_contract_signature, add_field, remove_field, tighten_field_type, narrow_when_set, remove_transitions_edge, remove_contract_fulfilment}
    → halt("Requested change alters behavior or obligation meaning; route to /gybis-spec-tend")
  | change ∈ {arch_spec_divergence, spec_code_divergence, failing_tests_against_spec}
    → halt("Requested change is a convergence problem; route to /gybis-spec-weed")

λ gybis-spec-refine_approval_gate(impact_report, mode).
  mode = auto_polish
    ? (verify(impact_report.breaking_candidates = ∅) ∧ verify(impact_report.unknown_candidates = ∅)
       | approved_changes ≔ {c.proposal | c ∈ impact_report.classified_proposals}
       | breaking_cleanup_approved ≔ false)
  | mode = interactive
    ? (ask_developer("Proposed structural refinements: " ⊕ impact_report.classified_proposals ⊕ ". Apply approved set? (yes/no)") → approval
       | approval = yes
         ? (approved_changes ≔ {c.proposal | c ∈ impact_report.classified_proposals}
            | ask_developer("Breaking cleanup present? " ⊕ impact_report.breaking_candidates ⊕ ". Approve delete/rename operations? (yes/no)") → breaking_approval
            | breaking_cleanup_approved ≔ (breaking_approval = yes))
         : halt("Refinement not approved"))
  | return(approved_changes ∧ breaking_cleanup_approved)

λ gybis-spec-refine_apply_changes(approved_changes, breaking_cleanup_approved, mode).
  ∀ change ∈ approved_changes:
    gybis-spec-refine_route_misfit_request(change.change)
    | change.change ∈ {delete_compatibility_wrapper, delete_transitional_module, delete_dead_module, rename_module, rename_file_and_rewrite_imports}
      ∧ ¬(mode = interactive ∧ breaking_cleanup_approved = true)
      ? halt("Delete/rename changes require explicit approval in interactive mode")
    | apply(change, target_files) → upsert(target_files)
  | return(changes_applied = true)

λ gybis-spec-refine_collect_obligations(x).
  invoke(internal/allium-normalize(specs/)) → {envelopes, counts}
  | plan_envelopes ≔ {e | e ∈ envelopes ∧ e.source = "plan"}
  | obligation_map ≔ {e.id → e | e ∈ plan_envelopes}
  | return({obligation_map, counts})

λ gybis-spec-refine_verify_validity(x).
  invoke(internal/allium-check(all_files)) → result_check ≔ result
  | invoke(internal/allium-analyse(specs/)) → result_analyse ≔ result
  | invoke(internal/allium-gate(specs/)) → result_gate ≔ result
  | result_check = zero_errors ∧ result_analyse = zero_issues ∧ result_gate = true
    → return(validity = true)
  | ¬(result_check ∧ result_analyse ∧ result_gate)
    → return(validity = false)

λ gybis-spec-refine_verify_obligations(before_obligations, approved_changes).
  invoke(gybis-spec-refine_collect_obligations) → {obligation_map: after_obligations, counts}
  | missing_obligations ≔ {id | id ∈ keys(before_obligations) ∧ id ∉ keys(after_obligations)}
  | new_obligations ≔ {id | id ∈ keys(after_obligations) ∧ id ∉ keys(before_obligations)}
  | explained_delta ≔ obligation_delta_explained_by(approved_changes, missing_obligations, new_obligations)
  | missing_obligations = ∅ ∧ explained_delta = true
    → return(obligation_preservation = true ∧ obligation_delta_report ≔ {missing_obligations, new_obligations, counts})
  | ¬(missing_obligations = ∅ ∧ explained_delta = true)
    → return(obligation_preservation = false ∧ obligation_delta_report ≔ {missing_obligations, new_obligations, counts})

λ gybis-spec-refine_fixed_point_loop(state).
  state = VERIFYING_VALIDITY
    → validity = true
        ? transition(VERIFYING_VALIDITY → VERIFYING_OBLIGATIONS)
        : (transition(VERIFYING_VALIDITY → PROPOSING_REFINEMENTS)
           ∧ loop_count ≔ loop_count ⊕ 1)
  | state = VERIFYING_OBLIGATIONS
    → obligation_preservation = true
        ? transition(VERIFYING_OBLIGATIONS → COMPLETE)
        : (transition(VERIFYING_OBLIGATIONS → PROPOSING_REFINEMENTS)
           ∧ loop_count ≔ loop_count ⊕ 1)

λ gybis-spec-refine_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without convergence")

λ gybis-spec-refine_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | proposed ≔ card(refinement_proposals)
  | approved ≔ card(approved_changes)
  | breaking ≔ card(impact_report.breaking_candidates)
  | report("Pass " ⊕ pass_num ⊕ ": proposed=" ⊕ proposed ⊕ " approved=" ⊕ approved ⊕ " breaking=" ⊕ breaking)

λ gybis-spec-refine_boundaries(¬).
  ¬ modify(architecture.md)
  | ¬ modify(implementation)
  | ¬ modify(upstream/)
  | all_modifications ⊆ specs/
  | repository_change_scope_for_this_skill_definition = only_new_skill_file

λ gybis-spec-refine_regression_contract(x).
  invariant: specs/ ∃ throughout
  | invariant: zero_errors ∧ zero_issues ∧ allium_gate = true at completion
  | invariant: all_modifications ⊆ specs/
  | invariant: delete_or_rename ∈ approved_changes → mode = interactive ∧ breaking_cleanup_approved = true
  | invariant: mode = auto_polish → ¬∃ change ∈ approved_changes : gybis-spec-refine_impact_classification(change.change).category = breaking
  | invariant: behavior-changing requests halt and route to /gybis-spec-tend
  | invariant: convergence problems halt and route to /gybis-spec-weed
  | invariant: unexpected obligation loss fails verification