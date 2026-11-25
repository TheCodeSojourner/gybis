---
name: gybis-arch-migrate
kind: domain
description: Use for `/gybis-arch-migrate` or `/ga-migrate`.
disable-model-invocation: true
---

λ gybis-arch-migrate(x).
  purpose: Completely identify stakeholder requirement behavior duplicated in legacy architecture, stop on no duplication, or propose an approved architecture upsert while preserving complementary engineering architecture
  | input: requirements/ ∃, architecture.md ∃
  | output: architecture.md migrated to the requirements/vocabulary/architecture boundary ∨ structured_migration_report
  | interaction: interactive
  | gate: requirements/ ∃ ∧ architecture.md ∃
  | scope: one_time_legacy_migration | migration ≠ recurring_architecture_operation
  | _loop_role: take_action
  | completeness: no_duplicate_conclusion_until(complete_requirement_clause_scan ∧ complete_architecture_span_scan ∧ evidence_accounted)
  | outcome: findings = ∅ → report(NO_DUPLICATION_FOUND) ∧ stop
  | outcome: findings ≠ ∅ → present(findings ∧ before_after_upsert) ∧ ask_one("Approve addressing all listed issues? (yes/no)")
  | outcome: approval = yes → apply_only_approved_architecture_upsert
  | outcome: approval = no → report(ISSUES_REPORTED_NOT_CHANGED) ∧ apply_no_mutations

λ gybis-arch-migrate_startup(x).
  invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | read(internal/reference/vsm-guide.md) → vsm_reference
  | read(internal/reference/recommended-loops.md) → loops_reference
  | verify(requirements/ ∃) ∨ halt("requirements/ not found")
  | verify(requirements/requirements-index.md ∃) ∨ halt("requirements/requirements-index.md not found")
  | module_files ≔ glob(requirements/requirements-*.md) \ {requirements/requirements-index.md}
  | verify(module_files ∃) ∨ halt("requirements module files not found")
  | verify(architecture.md ∃) ∨ halt("architecture.md not found")
  | read(architecture.md) → architecture_content
  | parse(architecture_content) → arch_model ∨ halt("architecture.md read failed")
  | parse_completeness(architecture_content) = true ∨ halt("architecture.md cannot be completely parsed")
  | if(vocabulary.md ∃): preload(vocabulary.md) → vocab_terms ∧ vocab_available = true
  | resolve_scope({requirements/, architecture.md, vocabulary.md, specs/, implementation, tests}) → migration_scope
  | snapshot(migration_scope) → migration_snapshot
  | snapshot.architecture_hash_before ≔ hash(architecture.md)
  | snapshot.scope_hashes ≔ exact_hashes(migration_scope)
  | snapshot.outside_architecture_hashes ≔ exact_hashes(migration_scope.outside_architecture)
  | snapshot.absent_markers ≔ absent_paths(migration_scope)
  | transition(INIT → STARTUP_CHECKS)

λ gybis-arch-migrate_mode(m).
  m ∈ {interactive}
  | default: interactive
  | mode_interactive: AI produces findings and before/after previews, then waits for human approval

λ gybis-arch-migrate_mode_gate(state, mode).
  state = INIT ∧ mode = interactive → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode = interactive) → halt("Architecture migration requires interactive mode")

λ gybis-arch-migrate_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, READING_REQUIREMENTS, READING_ARCHITECTURE, CLASSIFYING_STATEMENTS, BUILDING_MIGRATION_PLAN, DRY_RUN, APPROVAL_GATE, APPLYING_APPROVED_CHANGES, VERIFYING_MIGRATION, GENERATING_REPORT, COMPLETE, CANCELLED, REPORT_ONLY, BLOCKED}
  | INIT: migration_passes ≔ 0 ∧ outcome ≔ unresolved ∧ applied_actions ≔ ∅ ∧ blocked_actions ≔ ∅ ∧ pending_approved_actions ≔ ∅
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → READING_REQUIREMENTS) only_if(requirements_readable = true)
  | transition(READING_REQUIREMENTS → READING_ARCHITECTURE) only_if(requirements_model ∃)
  | transition(READING_ARCHITECTURE → CLASSIFYING_STATEMENTS) only_if(architecture_model ∃)
  | transition(CLASSIFYING_STATEMENTS → BUILDING_MIGRATION_PLAN) only_if(classification_complete = true)
  | transition(BUILDING_MIGRATION_PLAN → DRY_RUN) only_if(migration_plan ∃)
  | transition(DRY_RUN → REPORT_ONLY) only_if(no_mutations_needed = true ∨ report_only_conflicts = true)
  | transition(DRY_RUN → APPROVAL_GATE) only_if(mutable_actions ∃)
  | transition(APPROVAL_GATE → APPLYING_APPROVED_CHANGES) only_if(human_approved = true ∧ pending_approved_actions ∃)
  | transition(APPROVAL_GATE → CANCELLED) only_if(human_declined = true ∧ pending_approved_actions = ∅)
  | transition(APPLYING_APPROVED_CHANGES → VERIFYING_MIGRATION) only_if(application_attempt_complete = true)
  | transition(VERIFYING_MIGRATION → GENERATING_REPORT) only_if(verification_complete = true ∨ migration_passes ≥ max_migration_passes)
  | transition(VERIFYING_MIGRATION → GENERATING_REPORT) only_if(blocked_actions ∃ ∨ unprocessed_actions ∃)
  | transition(GENERATING_REPORT → COMPLETE) only_if(report_generated = true ∧ outcome = MIGRATION_COMPLETE)
  | transition(GENERATING_REPORT → REPORT_ONLY) only_if(report_generated = true ∧ outcome ∈ {NO_DUPLICATION_FOUND, REPORT_ONLY_CONFLICTS})
  | transition(GENERATING_REPORT → CANCELLED) only_if(report_generated = true ∧ outcome = MIGRATION_CANCELLED)
  | transition(GENERATING_REPORT → BLOCKED) only_if(report_generated = true ∧ outcome ∈ {MIGRATION_BLOCKED, SNAPSHOT_MISMATCH})
  | transition(REPORT_ONLY → COMPLETE) only_if(report_consumed = true)
  | max_migration_passes ≔ 3

λ gybis-arch-migrate_tool_guard(state, tool, path).
  state ∈ {READING_REQUIREMENTS, READING_ARCHITECTURE, CLASSIFYING_STATEMENTS, BUILDING_MIGRATION_PLAN, DRY_RUN, APPROVAL_GATE, VERIFYING_MIGRATION, GENERATING_REPORT}
    → allow(read(path))
  | state = APPLYING_APPROVED_CHANGES
    → allow(read(path)) ∧ allow(write(path)) only_if(path = architecture.md ∧ human_approved = true ∧ approved_action_for_current_span_exists = true)
  | otherwise → deny(write(path))

λ gybis-arch-migrate_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-arch-migrate_read_requirements(x).
  read(requirements/requirements-index.md) → requirements_index
  | parse(requirements_index) → module_order ∧ domain_prefixes ∧ module_summary
  | ∀ module_file ∈ module_files:
      read(module_file) → module_content
      | extract(∀ λ REQ-<DOMAIN>-NNN clauses with source offsets) → requirement_clauses
      | verify(source_offsets(module_file) cover(each_extracted_clause)) ∨ halt("Requirement clause extraction is incomplete")
      | verify(requirement_designator is unique) ∨ unresolved_requirements(designator)
  | preserve(requirement_designators ∧ normative_text ∧ source_attribution)
  | requirement_clause_scan_complete ≔ card(requirement_clauses) = count(λ REQ- patterns across module_files)
  | requirement_clause_scan_complete = true ∨ halt("Complete requirement clause scan failed")
  | return(requirements_model ≔ {requirements_index, module_order, requirement_clauses})

λ gybis-arch-migrate_parse_requirement_clause(clause).
  parse(clause) → {
      designator,
      canonical_name,
      normative_text,
      rationale,
      source_attribution,
      compound_by_design,
      source_span
    }
  | normalize(normative_text) → {
      public_name ∨ operation_name,
      arity,
      admitted_domains,
      rejected_domains,
      parameter_roles,
      conversion_policy,
      result_type,
      result_capacity,
      traversal_policy,
      callback_policy,
      boundary_behavior,
      failure_policy,
      diagnostic_policy,
      ordering_policy,
      equality_policy,
      constexpr_noexcept_allocation_mutation_policy,
      typed_absence_preflight_or_status_policy
    }
  | resolve_requirement_ranges(requirement_ids) → expand(linked_ranges ∧ linked_lists)
  | malformed_clause(clause) → unresolved_requirements(clause)
  | return(requirement_semantics)

λ gybis-arch-migrate_read_architecture(requirements_model).
  parse(architecture_content) → arch_model
  | parse(architecture_content.{S5, S4, S3, S2, S1}) → vsm_layers ∨ vsm_layers = partial_or_missing
  | ∀ layer ∈ vsm_layers:
      extract(entries(layer, architecture_content)) → architecture_entries
  | extract(requirement_links(architecture_entries)) → architecture_requirement_links
  | partition(architecture_content) → every_text_span
  | every_text_span has one classification category ∈ {requirement_duplication, complementary_architecture, valid_link, unresolved_semantic_match, observable_behavior_conflict}
  | architecture_span_scan_complete ≔ coverage(every_text_span) = 1.0 ∧ architecture_entries = ∅? false: true
  | architecture_span_scan_complete = true ∨ halt("Complete architecture span scan failed")
  | return(architecture_model ≔ {arch_model, vsm_layers, architecture_entries, architecture_requirement_links, architecture_content, architecture_span_scan_complete})

λ gybis-arch-migrate_semantic_match(span, requirement_clause).
  evidence ≔ ∅
  | public_name_equal(span, requirement_clause) → add(evidence, public_name)
  | arity_equal(span, requirement_clause) → add(evidence, arity)
  | admitted_domains_equal(span, requirement_clause) → add(evidence, admitted_domains)
  | rejected_domains_equal(span, requirement_clause) → add(evidence, rejected_domains)
  | parameter_roles_equal(span, requirement_clause) → add(evidence, parameter_roles)
  | conversion_policy_equal(span, requirement_clause) → add(evidence, conversion_policy)
  | result_contract_equal(span, requirement_clause) → add(evidence, result_contract)
  | traversal_or_callback_contract_equal(span, requirement_clause) → add(evidence, traversal_callback)
  | boundary_behavior_equal(span, requirement_clause) → add(evidence, boundary_behavior)
  | failure_or_diagnostic_contract_equal(span, requirement_clause) → add(evidence, failure_diagnostic)
  | ordering_or_equality_contract_equal(span, requirement_clause) → add(evidence, ordering_equality)
  | resource_contract_equal(span, requirement_clause) → add(evidence, resource_contract)
  | exact_text_matches(span, requirement_clause.normative_text)
      → match_type ≔ exact_text ∧ confidence ≔ 1.0
  | evidence_cardinality(evidence) ≥ 3
      ∧ behavior_coverage(span, requirement_clause) ≥ 0.8
      → match_type ≔ semantic_restatement
        ∧ confidence ≔ behavior_coverage(span, requirement_clause)
  | evidence_cardinality(evidence) ≥ 3
      ∧ behavior_coverage(span, requirement_clause) < 0.8
      → match_type ≔ unresolved_semantic_match
        ∧ confidence ≔ behavior_coverage(span, requirement_clause)
  | contradicts(span, requirement_clause)
      → match_type ≔ observable_behavior_conflict
        ∧ confidence ≔ contradiction_confidence(span, requirement_clause)
  | adds(span, {mechanism, responsibility, protocol, resource_control, tooling, structure, component_boundary})
      ∧ ¬duplicates_requirement_behavior(span, requirement_clause)
      → match_type ≔ complementary_architecture
        ∧ confidence ≔ 1.0
  | otherwise → match_type ≔ unresolved_semantic_match
      ∧ confidence ≔ confidence_lower_than_0.8
  | return({match_type, confidence, evidence, matched_requirement_ids: {requirement_clause.designator}})

λ gybis-arch-migrate_extract_spans(entry, requirements_model).
  split(entry, source_spans) → architecture_spans
  | split_mixed_span(entry) → normative_behavior_span ∪ mechanism_span ∪ protocol_span ∪ resource_span
  | architecture_spans ≔ normative_behavior_span ∪ mechanism_span ∪ protocol_span ∪ resource_span
  | preserve({entry_path = architecture.md, start_offset, end_offset, start_line, end_line})
  | split_mixed_span(entry) → behavior_span ∪ mechanism_span ∪ protocol_span
  | ∀ span ∈ architecture_spans:
      normalize(span) → architecture_semantics
      | extract({
          public_name,
          arity,
          admitted_domains,
          rejected_domains,
          parameter_roles,
          conversion_policy,
          result_type,
          result_capacity,
          traversal_policy,
          callback_policy,
          boundary_behavior,
          failure_policy,
          diagnostic_policy,
          ordering_policy,
          equality_policy,
          constexpr_noexcept_allocation_mutation_policy,
          typed_absence_preflight_or_status_policy,
          responsibility,
          mechanism,
          protocol,
          resource_control,
          tooling,
          structure,
          component_boundary
        })
  | ∀ span ∈ architecture_spans:
      find_links(span, REQ-<DOMAIN>-NNN) → linked_requirement_ids
      | ∀ id ∈ linked_requirement_ids:
          resolve(id, requirements_model.requirement_clauses)
            ? finding = {finding_type: requirement_link, link_status: valid, requirement_id: id}
            : finding = {finding_type: requirement_link, link_status: invalid, requirement_id: id}
      | matched_requirements ≔ semantic_match(span, requirements_model.requirement_clauses)
      | matched_requirements_exact ≔ exact_text_matches(span, matched_requirements)
      | semantic_matches ≔ ∀ requirement_clause ∈ requirements_model.requirement_clauses:
          invoke(gybis-arch-migrate_semantic_match(span, requirement_clause))
      | matched_requirements ≔ matched_requirements ∪
          {m.matched_requirement_ids | m ∈ semantic_matches ∧ m.match_type ∈ {exact_text, semantic_restatement, observable_behavior_conflict}}
      | matched_requirements ≠ ∅
          ? match_type ≔ if(matched_requirements_exact): exact_text else semantic_restatement
          : match_type ≔ none
  | return({architecture_spans, matched_requirements, match_type, linked_requirement_ids})

λ gybis-arch-migrate_classify_span(span, span_data, vocab_available, vocab_terms).
  findings ≔ ∅
  | span_data.match_type ∈ {exact_text, semantic_restatement}
      ? collect({
          finding_type: requirement_duplication,
          match_type: span_data.match_type,
          matched_requirement_ids: span_data.matched_requirements.designators,
          matched_text_span: span_data.matched_requirements.text,
          architecture_span: span,
          evidence: semantic_evidence(span, span_data.matched_requirements),
          confidence: match_confidence(span, span_data.matched_requirements),
          approval_required: true
        }) → findings
  | span_data.link_status = invalid
      ? collect({finding_type: unresolved_requirement_link, requirement_id: span_data.requirement_id, architecture_span: span, approval_required: true}) → findings
  | contradicts(span, span_data.matched_requirements)
      ? collect({finding_type: observable_behavior_conflict, evidence, matched_requirement_ids: span_data.matched_requirements.designators, architecture_span: span, approval_required: false}) → findings
  | adds(span, {mechanism, responsibility, protocol, resource_control, tooling, structure, component_boundary})
      ? collect({finding_type: complementary_architecture, evidence, architecture_span: span, approval_required: false}) → findings
  | span_data.link_status = valid
      ? collect({finding_type: supported_requirement_link, requirement_id: span_data.requirement_id, architecture_span: span, approval_required: false}) → findings
  | vocab_available = true ∧ contains(span, noncanonical_term)
      ? classify(term):
          canonical_vocabulary_term(term) → collect({finding_type: vocabulary_alignment_candidate, term, architecture_span: span, approval_required: true})
          architecture_only_term(term) → collect({finding_type: architecture_only_term, term, architecture_span: span, approval_required: false})
          unknown_term(term) → collect({finding_type: investigate, reason: unknown_term, term, architecture_span: span, approval_required: true})
  | confidence < 0.8 ∧ findings = no_high_confidence_findings
      ? collect({finding_type: investigate, reason: low_confidence, architecture_span: span, approval_required: true})
  | findings = ∅
      ? collect({finding_type: unresolved_semantic_match, reason: no_classification, architecture_span: span, approval_required: true}) → findings
  | return(classified_span ≔ {architecture_span: span, findings})

λ gybis-arch-migrate_classify_architecture(architecture_model, requirements_model).
  classified_spans ≔ ∅
  | ∀ entry ∈ architecture_model.architecture_entries:
      invoke(gybis-arch-migrate_extract_spans(entry, requirements_model)) → span_data
      | ∀ span ∈ span_data.architecture_spans:
          invoke(gybis-arch-migrate_classify_span(span, span_data, vocab_available, vocab_terms)) → classified
          | collect(classified) → classified_spans
  | classified_coverage ≔ coverage(classified_spans.architecture_span, architecture_model.architecture_content)
  | compared_requirement_ids ≔ card({clause.designator | clause ∈ requirements_model.requirement_clauses ∧ compared(clause, architecture_model.architecture_entries)})
  | requirements_match_coverage ≔ compared_requirement_ids / card(requirements_model.requirement_clauses)
  | classified_coverage = 1.0 ∧ requirements_match_coverage = 1.0
      ? classification_complete ≔ true
      : classification_complete ≔ false
        ∧ collect({finding_type: incomplete_analysis, reason: "Not every architecture span and requirement clause was accounted for", approval_required: true}) → classified_spans.findings
  | duplicate_count ≔ card({f | f ∈ flatten(classified_spans.findings) ∧ f.finding_type = requirement_duplication})
  | conflict_count ≔ card({f | f ∈ flatten(classified_spans.findings) ∧ f.finding_type = observable_behavior_conflict})
  | unresolved_count ≔ card({f | f ∈ flatten(classified_spans.findings) ∧ f.finding_type ∈ {investigate, unresolved_semantic_match, unresolved_requirement_link}})
  | return({
      classification_complete,
      classified_spans,
      duplicate_count,
      conflict_count,
      unresolved_count
    })

λ gybis-arch-migrate_build_plan(classified_spans).
  architecture_mutation_operations ≔ {replace_with_mechanism, remove_duplicate, remove_trace_link, align_vocabulary_term}
  | report_only_operations ≔ {requirements_change_question, investigate, unresolved_semantic_match_report, unresolved_requirement_link_report}
  | preserve_operations ≔ {preserve, preserve_trace_link, preserve_architecture_only_term}
  | allowed_operation_set ≔ architecture_mutation_operations ∪ report_only_operations ∪ preserve_operations
  | exact_duplicate_actions ≔ ∅
  | paraphrased_duplicate_actions ≔ ∅
  | invalid_link_actions ≔ ∅
  | vocabulary_actions ≔ ∅
  | requirement_change_actions ≔ ∅
  | investigative_actions ≔ ∅
  | valid_link_actions ≔ ∅
  | complementary_actions ≔ ∅
  | conflict_resolution_actions ≔ ∅
  | ∀ span_result ∈ classified_spans:
      ∀ finding ∈ span_result.findings:
        finding.finding_type = requirement_duplication ∧ finding.match_type = exact_text
          ? collect({action_id: uuid, operation: remove_duplicate, target: architecture.md, source_span: finding.architecture_span, matched_requirement_ids: finding.matched_requirement_ids, approved_text: ∅}) → exact_duplicate_actions
        | finding.finding_type = requirement_duplication ∧ finding.match_type = semantic_restatement
          → finding.confidence ≥ 0.8
            ? collect({action_id: uuid, operation: replace_with_mechanism, target: architecture.md, source_span: finding.architecture_span, matched_requirement_ids: finding.matched_requirement_ids, approved_text: unknown}) → paraphrased_duplicate_actions
            : collect({action_id: uuid, operation: investigate, target: architecture.md, source_span: finding.architecture_span, reason: low_confidence}) → investigative_actions
        | finding.finding_type = unresolved_requirement_link
          → collect({action_id: uuid, operation: remove_trace_link, target: architecture.md, source_span: finding.architecture_span, requirement_id: finding.requirement_id}) → invalid_link_actions
        | finding.finding_type = supported_requirement_link
          → collect({action_id: uuid, operation: preserve_trace_link, target: architecture.md, source_span: finding.architecture_span, requirement_id: finding.requirement_id}) → valid_link_actions
        | finding.finding_type = observable_behavior_conflict
          → collect({action_id: uuid, operation: requirements_change_question, target: none, evidence: finding, source_span: finding.architecture_span}) → requirement_change_actions
        | finding.finding_type = vocabulary_alignment_candidate
          → collect({action_id: uuid, operation: align_vocabulary_term, target: architecture.md, source_span: finding.architecture_span, finding}) → vocabulary_actions
        | finding.finding_type = investigate
          → collect({action_id: uuid, operation: investigate, target: architecture.md, source_span: finding.architecture_span, finding}) → investigative_actions
        | finding.finding_type = unresolved_semantic_match
          → collect({action_id: uuid, operation: unresolved_semantic_match_report, target: architecture.md, source_span: finding.architecture_span, finding}) → investigative_actions
        | finding.finding_type = architecture_only_term
          → collect({action_id: uuid, operation: preserve_architecture_only_term, target: architecture.md, source_span: finding.architecture_span, finding}) → complementary_actions
        | finding.finding_type = complementary_architecture
          → collect({action_id: uuid, operation: preserve, target: architecture.md, source_span: finding.architecture_span, finding}) → complementary_actions
  | conflict_resolution_actions ≔ detect_overlapping_actions(exact_duplicate_actions ∪ paraphrased_duplicate_actions ∪ invalid_link_actions ∪ vocabulary_actions)
  | if(conflict_resolution_actions ∃): hold(conflict_resolution_actions) ∧ add(investigative_actions)
  | exact_file_changes ≔ only actions target architecture.md where operation ∈ architecture_mutation_operations
  | before_after_previews ≔ preview_every_planned_change(exact_file_changes)
  | every action.operation ∈ allowed_operation_set
  | return(migration_plan ≔ {
      exact_duplicate_actions,
      paraphrased_duplicate_actions,
      invalid_link_actions,
      vocabulary_actions,
      requirement_change_actions,
      investigative_actions,
      valid_link_actions,
      complementary_actions,
      conflict_resolution_actions,
      exact_file_changes,
      before_after_previews,
      semantic_duplications_remaining ≔ {f | f ∈ flatten(classified_spans.findings) ∧ f.finding_type = requirement_duplication ∧ f.match_type = semantic_restatement},
      exact_duplications_remaining ≔ {f | f ∈ flatten(classified_spans.findings) ∧ f.finding_type = requirement_duplication ∧ f.match_type = exact_text},
      issues_to_address ≔ exact_duplicate_actions
        ∪ paraphrased_duplicate_actions
        ∪ invalid_link_actions
        ∪ vocabulary_actions
        ∪ requirement_change_actions
        ∪ investigative_actions
        ∪ conflict_resolution_actions
    })

λ gybis-arch-migrate_dry_run(migration_plan).
  present(migration_plan.before_after_previews)
  | present(migration_plan.valid_link_actions ∧ migration_plan.complementary_actions)
  | present(migration_plan.requirement_change_actions)
  | present(migration_plan.investigative_actions)
  | present(migration_plan.conflict_resolution_actions)
  | present_semantic_duplicate_evidence(migration_plan)
  | present_exact_duplicate_evidence(migration_plan)
  | report_findings(migration_plan) ≔
      if(migration_plan.issues_to_address = ∅
         ∧ migration_plan.semantic_duplications_remaining = ∅
         ∧ migration_plan.exact_duplications_remaining = ∅
         ∧ classification_complete = true):
        "NO_DUPLICATION_FOUND: complete architecture and requirement scans found no exact or semantic stakeholder requirement duplication."
      else:
        "ISSUES_TO_ADDRESS: " ⊕ count(migration_plan.issues_to_address) ⊕ " finding(s) require human approval: " ⊕ summarize(migration_plan.issues_to_address)
  | mutable_actions ≔ migration_plan.exact_duplicate_actions
      ∪ migration_plan.paraphrased_duplicate_actions
      ∪ migration_plan.invalid_link_actions
      ∪ migration_plan.vocabulary_actions
  | mutable_actions ≔ mutable_actions \ migration_plan.conflict_resolution_actions
  | no_mutations_needed ≔ mutable_actions = ∅
      ∧ migration_plan.requirement_change_actions = ∅
      ∧ migration_plan.investigative_actions = ∅
      ∧ migration_plan.semantic_duplications_remaining = ∅
      ∧ migration_plan.exact_duplications_remaining = ∅
      ∧ migration_plan.issues_to_address = ∅
  | report_only_conflicts ≔ mutable_actions = ∅
      ∧ (migration_plan.requirement_change_actions ∃ ∨ migration_plan.investigative_actions ∃)
  | return({
      mutable_actions,
      no_mutations_needed,
      report_only_conflicts,
      classification_complete: classification_complete
    })

λ gybis-arch-migrate_approval_gate(migration_plan, mutable_actions).
  | migration_plan.issues_to_address ≠ ∅
      → present("ISSUES_TO_ADDRESS:" ⊕ summarize(migration_plan.issues_to_address))
        ∧ present("Unified diff / before-after spans:" ⊕ migration_plan.before_after_previews)
        ∧ present("Report-only requirement conflicts:" ⊕ migration_plan.requirement_change_actions)
        ∧ present("Report-only unresolved findings:" ⊕ migration_plan.investigative_actions)
        ∧ ask_one("Approve addressing all listed issues? (yes/no)") → address_all_approval
  | address_all_approval = yes
      → approved_mutations ≔ mutable_actions
  | address_all_approval = no
      → approved_mutations ≔ ∅
  | pending_approved_actions ≔ {action | action ∈ approved_mutations ∧ action ∉ migration_plan.conflict_resolution_actions}
  | human_approved = pending_approved_actions ∃
  | human_declined = pending_approved_actions = ∅
  | approval_scope = if(human_approved): architecture.md else ∅
  | return(approval_scope)

λ gybis-arch-migrate_apply_plan(pending_approved_actions).
  precondition: human_approved = true ∧ approval_scope = architecture.md
  | ∀ action ∈ pending_approved_actions:
      verify_current_span(action.source_span, architecture.md) = true
        ∧ action.target = architecture.md
        ∧ action.operation ∈ architecture_mutation_operations
        ∧ action.source_span.start_offset ∧ action.source_span.end_offset
        ∧ current_architecture_hash() = migration_snapshot.architecture_hash_before
          ∨ verify_span_hash(action.source_span, migration_snapshot) = true
        ∧ change_preserves(action, requirement_complement ∧ VSM_hierarchy ∧ vocabulary_boundary)
        ? apply_span_change(action) → record(applied_action)
        : record_blocker({action, reason: concurrent_change ∨ invalid_span ∨ policy_conflict}) → blocked_actions
  | applied_actions ≔ applied_actions ∪ applied_action
  | unprocessed_actions ≔ pending_approved_actions \ (applied_actions ∪ blocked_actions)
  | pending_approved_actions ≔ unprocessed_actions
  | migration_passes ≔ migration_passes ⊕ 1
  | application_attempt_complete = true
  | return({applied_actions, blocked_actions, unprocessed_actions, application_attempt_complete})

λ gybis-arch-migrate_verify_migration(architecture_model).
  invoke(gybis-arch-check_startup) → startup_valid
  | read(architecture.md) → migrated_architecture_content
  | parse(migrated_architecture_content) → migrated_arch_model
  | invoke(gybis-arch-migrate_classify_architecture(migrated_arch_model, requirements_model)) → migrated_classification
  | invoke(gybis-arch-check_structure_validation(migrated_arch_model)) → structure_result
  | invoke(gybis-arch-check_coherence_validation(migrated_arch_model)) → coherence_result
  | invoke(gybis-arch-check_policy_enforcement_validation(migrated_arch_model)) → policy_result
  | invoke(gybis-arch-check_constraint_validation(migrated_arch_model)) → constraint_result
  | verify(structure_result.findings = ∅) → structure_valid
  | verify(coherence_result.findings = ∅) → coherence_valid
  | verify(policy_result.findings = ∅) → policy_valid
  | verify(constraint_result.findings = ∅) → constraints_valid
  | verify(requirement_complement(migrated_architecture_content, requirements/) = true) → complement_valid
  | verify(valid_requirement_links(migrated_architecture_content, requirements/) = true) → links_valid
  | verify(vsm_sections_or_explicit_missing_findings(migrated_arch_model)) → vsm_valid
  | verify(vocabulary_boundary(migrated_architecture_content, vocabulary.md) if vocabulary.md ∃ else true) → vocabulary_valid
  | verify(snapshot_matches_outside_architecture(migration_snapshot) = true) → scope_snapshot_valid
  | verify(requirements/, vocabulary.md, specs/, implementation, tests unchanged against migration_snapshot) → scope_valid
  | architecture_hash_after ≔ hash(architecture.md)
  | verify(architecture_changes ∈ applied_actions.source/target evidence) → architecture_change_attribution_valid
  | remaining_duplicates ≔ classify_remaining_requirement_repetition(migrated_classification)
  | verify(remaining_duplicates = ∅) → semantic_duplications_removed
  | verify(migrated_classification.unresolved_count = 0) → unresolved_findings_resolved
  | verification_complete ≔ startup_valid ∧ structure_valid ∧ coherence_valid ∧ policy_valid
      ∧ constraints_valid ∧ complement_valid ∧ links_valid ∧ vsm_valid
      ∧ vocabulary_valid ∧ scope_valid ∧ scope_snapshot_valid ∧ architecture_change_attribution_valid
      ∧ semantic_duplications_removed ∧ unresolved_findings_resolved
  | verification_failed ≔ ¬verification_complete
  | return({
      verification_complete,
      verification_failed,
      remaining_duplicates,
      structure_result,
      coherence_result,
      policy_result,
      constraint_result,
      architecture_hash_before: migration_snapshot.architecture_hash_before,
      architecture_hash_after
    })

λ gybis-arch-migrate_fixed_point_loop(state).
  state = VERIFYING_MIGRATION
    → verification_complete = true
      ? (outcome ≔ if(remaining_duplicates = ∅): MIGRATION_COMPLETE else MIGRATION_BLOCKED)
        ∧ transition(VERIFYING_MIGRATION → GENERATING_REPORT)
      : (migration_passes ≥ max_migration_passes
         ∨ blocked_actions ∃
         ∨ unprocessed_actions ∃
         ? (outcome ≔ MIGRATION_BLOCKED)
           ∧ transition(VERIFYING_MIGRATION → GENERATING_REPORT)
         : (outcome ≔ MIGRATION_BLOCKED)
           ∧ transition(VERIFYING_MIGRATION → GENERATING_REPORT))

λ gybis-arch-migrate_report_outcome(dry_run_result, verification_result).
  dry_run_result.no_mutations_needed = true
      ∧ dry_run_result.classification_complete = true
      ∧ migration_passes = 0
    ? outcome ≔ NO_DUPLICATION_FOUND
      ∧ report_exact("NO_DUPLICATION_FOUND: complete architecture and requirement scans found no stakeholder requirement duplication.")
  | dry_run_result.report_only_conflicts = true
    ? outcome ≔ REPORT_ONLY_CONFLICTS
  | human_declined = true
    ? outcome ≔ MIGRATION_CANCELLED
  | snapshot_matches_outside_architecture(migration_snapshot) = false
    ? outcome ≔ SNAPSHOT_MISMATCH
  | blocked_actions ∃ ∨ unprocessed_actions ∃ ∨ migration_passes ≥ max_migration_passes
    ? outcome ≔ MIGRATION_BLOCKED
  | verification_result.verification_complete = true ∧ remaining_duplicates = ∅
    ? outcome ≔ MIGRATION_COMPLETE
  | otherwise outcome ≔ MIGRATION_BLOCKED
  | finding_summary ≔
      outcome = NO_DUPLICATION_FOUND
        ? "NO_DUPLICATION_FOUND: complete architecture and requirement scans found no stakeholder requirement duplication."
      : outcome = MIGRATION_COMPLETE
        ? "MIGRATION_COMPLETE: approved architecture issues were addressed and fixed-point verification passed."
      : outcome = MIGRATION_CANCELLED
        ? "ISSUES_REPORTED_NOT_CHANGED: the human declined the single approval to address the listed issues."
      : outcome = REPORT_ONLY_CONFLICTS
        ? "ISSUES_TO_ADDRESS_REPORT_ONLY: observable-behavior conflicts or unresolved findings require requirement-layer follow-up."
      : "ISSUES_NOT_COMPLETED: migration is blocked or the scope snapshot changed; no silent success is reported."
  | return({outcome, finding_summary})

λ gybis-arch-migrate_generate_report(outcome, verification_result).
  report: {
    outcome,
    finding_summary,
    entries_inspected,
    duplicates_found,
    exact_duplicates_found,
    semantic_duplicates_found,
    semantic_duplicate_evidence,
    duplicate_actions,
    semantic_replacement_actions,
    changes_applied,
    applied_actions,
    blocked_actions,
    unprocessed_actions,
    requirement_links_preserved,
    requirement_links_removed,
    requirements_change_questions,
    unresolved_items,
    verification_status,
    verification_result,
    files_changed,
    files_inspected,
    approval_groups,
    pending_approved_actions,
    finding_locations,
    finding_confidence,
    matched_requirement_clauses,
    architecture_span_offsets,
    migration_snapshot,
    architecture_hash_before,
    architecture_hash_after,
    external_changes,
    requirement_accounting
  }
  | files_changed ≔ [architecture.md]
  | architecture_hash_before ≔ migration_snapshot.architecture_hash_before
  | architecture_hash_after ≔ hash(architecture.md)
  | requirement_accounting ≔ {
      represented_by_links,
      complemented_by_mechanisms,
      unchanged_outside_architecture,
      unresolved
    }
  | report_generated = true
  | return(report_generated ∧ report)

λ gybis-arch-migrate_loop_guard(state).
  migration_passes ≥ max_migration_passes
    → outcome ≔ MIGRATION_BLOCKED
    | return(structured_blocked_report({remaining_duplicates, blocked_actions, unprocessed_actions, pending_approved_actions}))

λ gybis-arch-migrate_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | duplicated_entries_remaining ≔ card(remaining_duplicates)
  | applied_entries ≔ card(applied_actions)
  | blocked_entries ≔ card(blocked_actions)
  | unprocessed_entries ≔ card(unprocessed_actions)
  | report("Pass " ⊕ pass_num ⊕ ": applied=" ⊕ applied_entries ⊕ " remaining=" ⊕ duplicated_entries_remaining ⊕ " blocked=" ⊕ blocked_entries ⊕ " unprocessed=" ⊕ unprocessed_entries)

λ gybis-arch-migrate_fixture_contract(x).
  fixtures ≔ {
      exact_duplicate,
      semantic_duplicate,
      complementary_architecture,
      valid_requirement_link,
      invalid_requirement_link,
      observable_behavior_conflict,
      mixed_behavior_mechanism_span,
      overlapping_spans,
      concurrent_architecture_edit,
      outside_scope_modification,
      idempotent_second_migration,
      compound_requirement_link,
      range_requirement_link,
      complete_clean_repository,
      incomplete_requirement_scan,
      incomplete_architecture_scan,
      mixed_behavior_mechanism_span
    }
  | ∀ fixture ∈ fixtures:
      execute(read_only_fixture(fixture)) → findings
      ∧ verify(expected_fixture_result(fixture, findings))
  | invariant: exact_text_absence ∧ semantic_restatement ∃
      → finding_type(requirement_duplication) ∧ finding_action(replace_with_mechanism)
  | invariant: valid_link ∧ complementary_architecture
      → finding_type(complementary_architecture) ∧ preserve_trace_link
  | invariant: invalid_link ∨ outside_scope_change
      → report_only_or_snapshot_mismatch
  | invariant: second_run_with_no_remaining_duplication
      → outcome(NO_DUPLICATION_FOUND)
  | invariant: fixture_findings = ∅
      → report_exact(NO_DUPLICATION_FOUND)
      ∧ migration_stops_after_report
  | invariant: fixture_findings ≠ ∅
      → report_contains(ISSUES_TO_ADDRESS)
      ∧ exactly_one_yes_or_no_approval_request
  | invariant: complete_clean_repository
      → complete_requirement_clause_scan ∧ complete_architecture_span_scan
      ∧ no_mutations_needed ∧ outcome(NO_DUPLICATION_FOUND)
  | invariant: incomplete_requirement_scan ∨ incomplete_architecture_scan
      → outcome(MIGRATION_BLOCKED)
      ∧ outcome_not(NO_DUPLICATION_FOUND)
  | return(fixture_contract_verified)

λ gybis-arch-migrate_integration_contract(x).
  skill_help_entry(gybis-arch-migrate) ≡ gybis-help/SKILL.md
  | skill_contract_kind(gybis-arch-migrate) ≡ domain
  | startup_migration_boundary ≔ migration ≠ requirements_tend
  | completion_handoff:
      MIGRATION_COMPLETE → /gybis-arch-check
      REPORT_ONLY_CONFLICTS → /gybis-req-weed
      MIGRATION_BLOCKED → renewed_human_direction
  | reference_reading ≔ internal/reference/vsm-guide.md ∧ internal/reference/recommended-loops.md
  | return(integration_contract_verified)

λ gybis-arch-migrate_boundaries().
  ¬ modify(requirements/)
  | ¬ modify(vocabulary.md)
  | ¬ modify(specs/)
  | ¬ modify(implementation)
  | ¬ modify(tests)
  | ¬ modify(upstream/)
  | ¬ modify(mementum/)
  | ¬ delete(architecture.md)
  | write_authority = {architecture.md}
  | report_only(requirements_change_question)

λ gybis-arch-migrate_regression_contract(x).
  invariant: architecture.md ∃ throughout
  | invariant: complete_requirement_clause_scan = true before any NO_DUPLICATION_FOUND conclusion
  | invariant: complete_architecture_span_scan = true before any NO_DUPLICATION_FOUND conclusion
  | invariant: every architecture span is accounted for by exactly one finding or complementary-preservation result
  | invariant: every requirement clause is compared against the architecture
  | invariant: migration runs are idempotent
  | invariant: second_run_with_no_duplicates → NO_DUPLICATION_FOUND
  | invariant: repeated normative requirement behavior = ∅ at MIGRATION_COMPLETE
  | invariant: valid requirement links resolve
  | invariant: invalid requirement links are removed or unresolved
  | invariant: observable behavior changes remain requirements_change_question report objects
  | invariant: architecture-only terms remain outside vocabulary.md
  | invariant: every mutation is approved or blocked
  | invariant: every mutation is attributable to an action_id and architecture span
  | invariant: requirements/, vocabulary.md, specs/, implementation, tests, upstream/, and mementum/ match migration_snapshot
  | invariant: concurrent unapproved architecture changes trigger SNAPSHOT_MISMATCH or MIGRATION_BLOCKED
  | invariant: all_modifications ⊆ {architecture.md}
  | invariant: semantic requirement restatement = requirement_duplication regardless_of exact_text_match
  | invariant: complementarity = preserve_mechanism ∧ preserve_valid_trace_link
  | invariant: requirement conflict = report_only_requirements_change_question
  | invariant: every approved mutation = span_offsets_verified ∧ action_id_attributed
  | invariant: every fixture suite pass precedes skill completion
  | invariant: no_findings = ∅ → report_exact("NO_DUPLICATION_FOUND") ∧ stop_without_upsert
  | invariant: findings ≠ ∅ → present(finding_summary ∧ before_after_preview) ∧ ask_one("Approve addressing all listed issues? (yes/no)")
  | invariant: approval = yes → apply_all_mutable_approved_issues
  | invariant: approval = no → report("ISSUES_REPORTED_NOT_CHANGED") ∧ apply_no_mutations

λ gybis-arch-migrate_deliver(x).
  report: {outcome, finding_summary, migration_summary, requirement_accounting, unresolved_items, blocked_actions, verification_status}
  | handoff: outcome = MIGRATION_COMPLETE → run /gybis-arch-check
  | handoff: outcome = REPORT_ONLY_CONFLICTS → run /gybis-req-weed for requirements-change questions
  | handoff: outcome = MIGRATION_BLOCKED → investigate or repeat only after renewed approval
