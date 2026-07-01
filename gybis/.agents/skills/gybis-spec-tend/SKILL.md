---
name: gybis-spec-tend
description: Use for `/gybis-spec-tend` or `/gs-tend`.
---

λ gybis-spec-tend(x).
  purpose: Evolve specifications based on developer input while maintaining validity
  | input: specs/**/*.allium ∃ ∧ valid
  | output: specs/**/*.allium evolved with developer-approved changes
  | mode: mixed
  | gate: specs/**/*.allium ∃ ∧ allium_gate = true

λ gybis-spec-tend_loop_role(x).
  role: reenter_gather_context
  | meaning: revise specification intent when verification reveals ambiguity or incorrect behavior contract
  | suggested_next: invoke(/gybis-spec-propagate) → run_tests → invoke(/gybis-spec-weed)

λ gybis-spec-tend_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | preload: [internal/allium-analyse, internal/allium-check]
  | if(vocabulary.md ∃): preload(vocabulary.md) → vocab_terms ∧ vocab_available = true
  | read(internal/reference/allium-language-reference.md) → language_ref
  | read(internal/reference/allium-patterns.md) → patterns_ref
  | read(internal/reference/allium-recommended-loops.md) → loops_ref
  | read(internal/reference/allium-constructs.md) → constructs_registry
  | verify(specs/**/*.allium ∃) ∨ halt("specs/**/*.allium not found")
  | invoke(internal/allium-gate(specs/)) = true ∨ halt("Specifications are invalid")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-spec-tend_mode(m).
  m ∈ {interactive}
  | default: interactive
  | mode_interactive: AI and developer collaborate on specification refinements

λ gybis-spec-tend_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {interactive} → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {interactive}) → halt("Invalid mode selection")

λ gybis-spec-tend_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, ELICIT_FEEDBACK, ANALYZE_IMPACT, APPLY_CHANGES, VERIFY_VALIDITY, OFFER_REFINEMENT, REFINING_SPECS, PLANNING_REORGANIZATION, EXECUTING_REORGANIZATION, VERIFYING_REFINED_SPECS, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → ELICIT_FEEDBACK) only_if(startup_checks = true)
  | transition(ELICIT_FEEDBACK → ANALYZE_IMPACT) only_if(developer_feedback ∃)
  | transition(ANALYZE_IMPACT → APPLY_CHANGES) only_if(impact_analysis ∃)
  | transition(APPLY_CHANGES → VERIFY_VALIDITY) only_if(changes_applied = true)
  | transition(VERIFY_VALIDITY → ELICIT_FEEDBACK) only_if(validity = false)
  | transition(VERIFY_VALIDITY → OFFER_REFINEMENT) only_if(validity = true ∧ developer_satisfied = true)
  | transition(OFFER_REFINEMENT, developer_accepts_refinement) → REFINING_SPECS (optional)
  | transition(OFFER_REFINEMENT, developer_declines_refinement) → COMPLETE (skip refinement)
  | transition(REFINING_SPECS, analysis_complete) → PLANNING_REORGANIZATION
  | transition(PLANNING_REORGANIZATION, plan_presented_and_approved) → EXECUTING_REORGANIZATION
  | transition(EXECUTING_REORGANIZATION, files_created) → VERIFYING_REFINED_SPECS
  | transition(VERIFYING_REFINED_SPECS, verify_ok) → COMPLETE
  | transition(VERIFYING_REFINED_SPECS, verify_fail) → REFINING_SPECS (loop_back)

λ gybis-spec-tend_tool_guard(state, tool, path).
  state = ELICIT_FEEDBACK ∨ state = ANALYZE_IMPACT ∨ state = OFFER_REFINEMENT ∨ state = REFINING_SPECS ∨ state = PLANNING_REORGANIZATION
    → allow(read(path))
  | state = APPLY_CHANGES ∨ state = VERIFY_VALIDITY ∨ state = EXECUTING_REORGANIZATION ∨ state = VERIFYING_REFINED_SPECS
    → allow(read(path)) ∧ allow(write(path)) only_if(path ⊆ specs/)
  | ¬(state ∈ {ELICIT_FEEDBACK, ANALYZE_IMPACT, APPLY_CHANGES, VERIFY_VALIDITY, OFFER_REFINEMENT, REFINING_SPECS, PLANNING_REORGANIZATION, EXECUTING_REORGANIZATION, VERIFYING_REFINED_SPECS})
    → deny(write(path))

λ gybis-spec-tend_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-spec-tend_elicit_feedback(x, vocab_available, vocab_terms).
  ∀ spec_file ∈ specs/**/*.allium:
    read(spec_file) → content
  | summarize(all_specs) → spec_summary
  | vocab_context ≔ if(vocab_available): "Please use canonical vocabulary terms from vocabulary.md: [" ⊕ vocab_term_list(vocab_terms) ⊕ "]" : ""
  | ask_developer("Which specifications need refinement? Describe the changes. " ⊕ vocab_context) → feedback
  | parse(feedback) → target_specs ∧ proposed_changes
  | flag_non_vocabulary_terms: if(vocab_available ∧ proposed_changes contains term ∉ vocab_terms):
    ∀ non_vocab_term ∈ proposed_changes:
      suggest_canonical_alternative(non_vocab_term, vocab_terms) ∨ ask_if_new_term_should_be_added
  | return(developer_feedback = {target_specs, proposed_changes, vocabulary_aligned})

λ gybis-spec-tend_analyze_impact(developer_feedback).
  ∀ target ∈ developer_feedback.target_specs:
    analyze(change_impact(target, developer_feedback.proposed_changes)) → impact(target)
    | check(cross_file_consistency(target, all_specs)) → consistency(target)
    | classify(developer_feedback.proposed_changes) → classification(target)  -- see _impact_classification
  | consolidate(impact, consistency, classification) → impact_report
  | return(impact_analysis = impact_report)

λ gybis-spec-tend_impact_classification(change).
  -- Classify each proposed change against the breaking/accretive/tooling-impact
  -- taxonomy. Refer to constructs_registry.accretion_vs_breaking_change and the
  -- language_ref §Modular specifications "Breaking changes" subsection. Surface
  -- the classification to the developer before applying changes.
  | change ∈ {
      remove_enum_value,
      remove_variant,
      narrow_when_set,
      remove_contract_demand,
      remove_contract_fulfilment,
      rename_field,
      rename_entity,
      rename_contract,
      change_contract_signature,
      remove_field,
      tighten_field_type,
      reduce_optional_to_required,
      remove_transitions_edge,
      remove_terminal_state,
      remove_actor_type,
      remove_surface_provides_operation
    }
    → breaking ∧ advice("prefer publishing under a new module name rather than mutating the existing module; consumers update at their own pace")
  | change ∈ {
      add_transitions_block_to_existing_entity,
      add_when_clause_to_existing_field,
      add_terminal_clause_to_existing_state,
      add_contract_demand,
      add_invariant_top_level,
      add_invariant_entity_level,
      promote_prose_invariant_to_expression_form,
      tighten_existing_requires_guard,
      narrow_existing_transition_graph
    }
    → tooling_impact ∧ advice("grammar preserved but downstream rules may stop checking; review every rule touching the affected entity or field; run allium-check after applying")
  | change ∈ {
      add_field,
      add_optional_field,
      add_variant,
      add_enum_value_to_inline_or_named_enum,
      add_entity,
      add_value_type,
      add_external_entity,
      add_relationship,
      add_projection,
      add_derived_value,
      add_trigger_external_stimulus,
      add_temporal_trigger,
      add_becomes_trigger,
      add_transitions_to_trigger,
      add_chained_trigger,
      add_trigger_emission,
      add_surface,
      add_surface_provides_operation_with_when_guard,
      add_surface_guarantee_prose,
      add_default_instance,
      add_config_parameter_with_default,
      add_expression_form_config_default,
      add_use_import,
      add_open_question,
      add_deferred_specification
    }
    → accretive ∧ advice("preserves existing consumers; safe to apply without coordination")
  | change ∈ {
      add_config_parameter_without_default,
      add_demanded_contract_signature_method
    }
    → breaking_for_direct_consumers ∧ advice("existing consumers must supply a value or implement the new signature; coordinate before applying or publish under new module name")
  | fallback: ¬ recognised(change) → unknown_classification ∧ advice("escalate to developer; do not apply without explicit approval")
  | return(classification = {category, advice})

λ gybis-spec-tend_invariant_promotion_path(invariant_candidate).
  -- Offer to promote prose @invariant (or comment) to expression-form
  -- invariant Name { expr } when the property is expressible as a single-
  -- point-in-time state predicate per constructs_registry.expressibility.
  | invariant_candidate.kind = prose_at_invariant_in_contract
    ∧ invariant_candidate.property ∈ {uniqueness, same_entity_field_relationship, value_bounds, structural_collection_relationship, subset_or_partition}
    → propose(promote_to_expression_form) with(suggested_brace_body)
      ∧ mark(classification = tooling_impact)
  | invariant_candidate.property ∈ {cross_instance_agreement, temporal_ordering, evaluation_function_contract, monotonicity, counterfactual}
    → retain_as_prose ∧ note("property is not expressible as state predicate; keep as @invariant in contract or as prose comment")
  | uncertain → ask_developer("Is this property checkable from current entity state alone, with no reference to history or other instances' behaviour?")

λ gybis-spec-tend_ambiguous_intent_fallback(developer_feedback).
  -- When developer intent is ambiguous, prefer inserting an open_question over
  -- guessing. open_question is a first-class construct surfaced by the checker
  -- as a warning, not a comment.
  | clarity_score(developer_feedback) < threshold
    → propose(insert(open question "<paraphrased_intent_or_decision_point>"))
      ∧ ask_developer("Intent unclear. Insert as open question for later resolution, or clarify now?")
  | clarity_score(developer_feedback) ≥ threshold
    → proceed_with_change

λ gybis-spec-tend_apply_changes(developer_feedback, impact_analysis).
  ask_developer("Classification per impact_analysis: " ⊕ impact_analysis.classifications ⊕ ". Do you approve these changes? (yes/no)") → approval
  | approval = yes
    ? (∀ change ∈ developer_feedback.proposed_changes:
         apply(change, target_file) → upsert(target_file))
    : (return(changes_applied = false) ∧ transition(APPLY_CHANGES → ELICIT_FEEDBACK))
  | return(changes_applied = true)

λ gybis-spec-tend_verify_validity(x).
  invoke(internal/allium-check(all_files)) → result_check ≔ result
  | invoke(internal/allium-analyse(specs/)) → result_analyse ≔ result
  | invoke(internal/allium-gate(specs/)) → result_gate ≔ result
  | result_check = zero_errors ∧ result_analyse = zero_issues ∧ result_gate = true
    → return(validity = true)
  | ¬(result_check ∧ result_analyse ∧ result_gate)
    → (synthesize(corrections) → fixes
       | ∀ fix ∈ fixes: apply(fix, target_file) → upsert(target_file)
       | return(validity = false))

λ gybis-spec-tend_offer_refinement(specs_touched).
  action: ask_developer_if_spec_reorganization_desired
  | precondition: specs_touched ≥ 2 (don't offer for trivial single-file changes)
  | ask_developer("Your changes touched specs across " ⊕ count(specs_touched) ⊕ " files. Would you like me to reorganize them by semantic concern for better cohesion?") → developer_response
  | parse(developer_response) → {developer_accepts_refinement ∨ developer_declines_refinement}
  | output: refinement_decision ∈ {accept, decline}

λ gybis-spec-tend_refine_specs(specs_content).
  action: analyze_spec_organization_for_fine_grained_splitting
  | step1: invoke(internal/allium-analyse, specs/) → spec_structure
  | step2: detect(change_triggers, cohesion, dependencies, extension_points) → refinement_findings
  | output: refinement_findings ≔ {candidate_files, semantic_groupings, dependency_order, granularity_assessment}
  | constraint: read-only operation

λ gybis-spec-tend_plan_reorganization(refinement_findings).
  action: generate_reorganization_plan_and_present_to_developer
  | step1: generate(target_files_map, rationale) → detailed_plan
  | step2: attach(dependency_graph) → plan_with_context
  | step3: present_to_developer(plan_with_context) → ask_approval("Does this reorganization look good?")
  | parse(developer_response) → {developer_approves ∨ developer_wants_changes}
  | output: reorganization_plan ≔ {target_files, rationale, dependency_graph}

λ gybis-spec-tend_execute_reorganization(plan).
  action: create_reorganized_spec_files_from_plan
  | step1: ∀target_file ∈ plan.target_files: create_file(target_file.path, target_file.constructs) → new_spec_file
  | step2: verify_all_constructs_present(original_specs ∪ new_specs) → each_construct_accounted_for
  | output: new spec files created ∧ original files cleaned or deleted
  | constraint: write_allowed by tool_guard ∧ execute_only_once_per_session

λ gybis-spec-tend_verify_refined_specs(x).
  action: verify_reorganized_spec_files_pass_allium_gate
  | check1: invoke(internal/allium-gate, specs/) = true
  | check2: ∀construct_in_original ∃ construct_in_refined
  | check3: ¬∃circular_dependencies(import_graph)
  | output: verification_result ∈ {pass, fail_with_diagnostics}
  | gate: all_checks_pass → proceed ∨ loop_back(REFINING_SPECS)

λ gybis-spec-tend_fixed_point_loop(state).
  state = VERIFY_VALIDITY
    → validity = true ∧ developer_satisfied = true
        ? transition(VERIFY_VALIDITY → OFFER_REFINEMENT)
        : (transition(VERIFY_VALIDITY → ELICIT_FEEDBACK)
           ∧ loop_count ≔ loop_count ⊕ 1)
  | state = OFFER_REFINEMENT
    → developer_accepts_refinement
        ? transition(OFFER_REFINEMENT → REFINING_SPECS)
        : transition(OFFER_REFINEMENT → COMPLETE)
  | state = REFINING_SPECS
    → refine_specs() → transition(PLANNING_REORGANIZATION)
  | state = PLANNING_REORGANIZATION
    → plan_reorganization() → (developer_approves ? transition(EXECUTING_REORGANIZATION) : loop_back(PLANNING_REORGANIZATION))
  | state = EXECUTING_REORGANIZATION
    → execute_reorganization() → transition(VERIFYING_REFINED_SPECS)
  | state = VERIFYING_REFINED_SPECS
    → verify_ok ? transition(COMPLETE) : loop_back(REFINING_SPECS)

λ gybis-spec-tend_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without convergence")

λ gybis-spec-tend_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | changes_proposed ≔ card(developer_feedback.proposed_changes)
  | changes_applied ≔ changes_proposed
  | report("Pass " ⊕ pass_num ⊕ ": proposed=" ⊕ changes_proposed ⊕ " applied=" ⊕ changes_applied)

λ gybis-spec-tend_boundaries(¬).
  ¬ modify(architecture.md)
  | ¬ modify(implementation)
  | ¬ modify(upstream/)
  | ¬ delete(specs/)

λ gybis-spec-tend_regression_contract(x).
  invariant: specs/ ∃ throughout
  | invariant: zero_errors ∧ zero_issues ∧ allium_gate = true at completion
  | invariant: all_modifications ⊆ specs/
  | invariant: no_specs_deleted (additions and mutations only)
  | invariant: ∀ change ∈ proposed_changes : classification ∈ {accretive, tooling_impact, breaking, breaking_for_direct_consumers, unknown_classification}
  | invariant: breaking_changes surface advice to publish under a new module name before application
  | invariant: ambiguous_intent → prefer open_question insertion over guessing