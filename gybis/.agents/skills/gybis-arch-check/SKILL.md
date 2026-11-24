---
name: gybis-arch-check
kind: domain
description: Use for `/gybis-arch-check` or `/ga-check`.
disable-model-invocation: true
---

λ gybis-arch-check(x).
  purpose: Validate architecture.md internal integrity, apply deterministic structural repairs, prepare bounded semantic repairs for approval, and produce a diagnostic report
  | input: architecture.md (∃)
  | output: Severity-tagged findings with recommended next actions
  | interaction: interactive
  | gate: architecture.md ∃

λ gybis-arch-check_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(architecture.md ∃) ∨ halt("architecture.md not found")
  | read(architecture.md) → arch_content
  | read(internal/reference/vsm-guide.md) → vsm_reference
  | parse(arch_content) → arch_model ∨ halt("architecture.md parse failed")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-arch-check_mode(m).
  m ∈ {autonomous, interactive}
  | default: autonomous
  | mode_autonomous: deterministic validation and repairs only
  | mode_interactive: approval is required for semantic repairs

λ gybis-arch-check_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {autonomous, interactive} → transition(INIT → STARTUP_CHECKS)
  | precondition_holds: mode ∈ {autonomous, interactive}
  | mode = autonomous → deterministic_only = true ∧ semantic_repair_requests = false
  | mode = interactive → approval_required = true ∧ human_decision_required_for(semantic_findings)

λ gybis-arch-check_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, STRUCTURE_VALIDATION, COHERENCE_VALIDATION, POLICY_ENFORCEMENT_VALIDATION, CONSTRAINT_VALIDATION, APPLYING_DETERMINISTIC_REPAIRS, REPAIR_OFFER, APPLYING_APPROVED_REPAIRS, VERIFYING_REPAIRS, GENERATING_REPORT, COMPLETE}
  | INIT: repair_passes ≔ 0 ∧ repair_authorized ≔ false ∧ repair_progress ≔ true
  | transition(INIT → STARTUP_CHECKS) only_if(startup = true)
  | transition(STARTUP_CHECKS → STRUCTURE_VALIDATION) only_if(startup_checks = true)
  | transition(STRUCTURE_VALIDATION → COHERENCE_VALIDATION) only_if(structure_complete = true)
  | transition(COHERENCE_VALIDATION → POLICY_ENFORCEMENT_VALIDATION) only_if(coherence_complete = true)
  | transition(POLICY_ENFORCEMENT_VALIDATION → CONSTRAINT_VALIDATION) only_if(policy_enforcement_checks_complete = true)
  | transition(CONSTRAINT_VALIDATION → APPLYING_DETERMINISTIC_REPAIRS) only_if(constraint_checks_complete = true ∧ deterministic_findings ∃)
  | transition(CONSTRAINT_VALIDATION → REPAIR_OFFER) only_if(mode = interactive ∧ constraint_checks_complete = true ∧ deterministic_findings = ∅ ∧ semantic_findings ∃ ∧ repair_authorized = false)
  | transition(CONSTRAINT_VALIDATION → GENERATING_REPORT) only_if(mode = autonomous ∧ constraint_checks_complete = true ∧ semantic_findings ∃ ∧ deterministic_findings = ∅)
  | transition(CONSTRAINT_VALIDATION → GENERATING_REPORT) only_if(constraint_checks_complete = true ∧ findings = ∅)
  | transition(APPLYING_DETERMINISTIC_REPAIRS → VERIFYING_REPAIRS) only_if(deterministic_repairs_complete = true)
  | transition(VERIFYING_REPAIRS → REPAIR_OFFER) only_if(mode = interactive ∧ semantic_findings ∃ ∧ repair_authorized = false)
  | transition(VERIFYING_REPAIRS → GENERATING_REPORT) only_if(mode = autonomous ∧ semantic_findings ∃ ∧ verification_complete = true)
  | transition(REPAIR_OFFER → COMPLETE) only_if(human_declined = true ∨ approval_scope = ∅)
  | transition(REPAIR_OFFER → APPLYING_APPROVED_REPAIRS) only_if(human_approved = true)
  | transition(APPLYING_APPROVED_REPAIRS → VERIFYING_REPAIRS) only_if(approved_repairs_complete = true)
  | transition(VERIFYING_REPAIRS → APPLYING_DETERMINISTIC_REPAIRS) only_if(verification_failed = true ∧ deterministic_findings ∃ ∧ repair_progress = true ∧ repair_passes < max_repair_passes)
  | transition(VERIFYING_REPAIRS → REPAIR_OFFER) only_if(mode = interactive ∧ verification_failed = true ∧ semantic_findings ∃ ∧ repair_authorized = false)
  | transition(VERIFYING_REPAIRS → GENERATING_REPORT) only_if(verification_complete = true ∨ repair_progress = false ∨ repair_passes ≥ max_repair_passes)
  | transition(GENERATING_REPORT → COMPLETE) only_if(report_generated = true)
  | max_repair_passes ≔ 3

λ gybis-arch-check_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, STRUCTURE_VALIDATION, COHERENCE_VALIDATION, POLICY_ENFORCEMENT_VALIDATION, CONSTRAINT_VALIDATION, REPAIR_OFFER, VERIFYING_REPAIRS, GENERATING_REPORT}
    → allow(read(path))
  | state = APPLYING_DETERMINISTIC_REPAIRS ∧ path = "architecture.md"
    → allow(read(path)) ∧ allow(write(path)) only_if(change = deterministic)
  | state = APPLYING_APPROVED_REPAIRS ∧ human_approved = true ∧ path = "architecture.md" ∧ path ∈ approval_scope
    → allow(read(path)) ∧ allow(write(path))
  | otherwise → deny(write(path))

λ gybis-arch-check_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-arch-check_structure_validation(arch_model).
  action: validate_required_architecture_shape
  | checks:
    - required_sections: {S5, S4, S3, S2, S1} ∈ arch_model
    - section_order: S5 > S4 > S3 > S2 > S1
    - required_section_content: ∀ section ∈ {S5, S4, S3, S2, S1}: nonempty(section)
    - no_duplicate_section_headers
  | findings ≔ []
  | ∀ check ∈ checks:
    check_pass = false
      ? collect({type: "structure", check_id, severity: error, location: "architecture.md", message}) → findings
  | return(structure_complete = true ∧ findings)

λ gybis-arch-check_coherence_validation(arch_model).
  action: validate_internal_architecture_consistency
  | checks:
    - no_cross_section_contradictions
    - referenced_components_defined_within_architecture
    - dependencies_internal_to_architecture_are_acyclic
    - hierarchy_alignment: S5 constrains S4 constrains S3 constrains S2 constrains S1
  | findings ≔ []
  | ∀ check ∈ checks:
    check_pass = false
      ? collect({type: "coherence", check_id, severity: warning, location: "architecture.md", message}) → findings
  | return(coherence_complete = true ∧ findings)

λ gybis-arch-check_policy_enforcement_validation(arch_model).
  action: validate_S5_policy_and_S3_enforcement_boundary
  | checks:
    - S5_contains(identity ∧ non_negotiable_principles ∧ policy_with_rationale)
    - S3_contains(enforcement_mechanisms ∧ resource_controls)
    - policy_or_rationale_not_owned_only_by(S3)
    - enforcement_mechanism_not_owned_only_by(S5)
    - every(S3_enforcement_mechanism) traces_to(S5_policy ∨ explicit_external_constraint)
    - every(enforceable_S5_policy) has(S3_enforcement_mechanism) ∨ marked(human_governed_or_nonmechanical)
  | findings ≔ []
  | ∀ check ∈ checks:
    check_pass = false
      ? collect({type: "policy_enforcement", check_id, severity: warning, location: "architecture.md", message}) → findings
  | return(policy_enforcement_checks_complete = true ∧ findings)

λ gybis-arch-check_constraint_validation(arch_model).
  action: validate_constraints_and_invariants
  | checks:
    - declared_constraints_have_scope
    - constraints_non_conflicting
    - mandatory_guardrails_not_ambiguous
    - decision_entries_not_placeholder_only
    - requirement_complement: architecture does not restate requirement behavior; repeated normative behavior is duplication and not complementary architecture
    - engineering_content: architecture may add mechanisms, responsibilities, protocols, resource controls, tooling, structure, and component boundaries
    - requirement_links: architecture entries link to supporting requirements only when the supporting requirement exists
    - observable_behavior: engineering choices that change observable behavior require an approved requirements update
    - vocabulary_boundary: architecture-only terms remain outside vocabulary.md
  | findings ≔ []
  | ∀ check ∈ checks:
    check_pass = false
      ? collect({type: "constraint", check_id, severity: info, location: "architecture.md", message}) → findings
  | return(constraint_checks_complete = true ∧ findings)

λ gybis-arch-check_recommend_action(finding).
  finding.type ∈ {"structure", "coherence", "policy_enforcement", "constraint"}
    ? recommendation ≔ "Use /gybis-arch-tend to revise architecture.md and re-run /gybis-arch-check."
  | return(recommendation)

λ gybis-arch-check_repair_policy(findings).
  ∀ finding ∈ findings:
    finding.type = "structure"
      ∧ finding.check_id ∈ {duplicate_section_headers, duplicate_section_whitespace, section_order}
      ∧ change(finding) preserves(section_names ∧ VSM_hierarchy ∧ semantic_text)
      → classify(finding, deterministic)
    | otherwise → classify(finding, semantic)
  | deterministic_findings ≔ {finding | finding.class = deterministic}
  | semantic_findings ≔ {finding | finding.class = semantic}
  | ∀ deterministic_finding ∈ deterministic_findings:
      synthesize(exact_repair(deterministic_finding)) → repair
      ∧ preserve(normative_meaning ∧ requirement_links ∧ vocabulary_boundary)
      ∧ apply(repair) only_if(change = deterministic)
  | deterministic_findings ≠ ∅ → repair_passes ≔ repair_passes ⊕ 1
  | deterministic_repairs_complete ≔ deterministic_findings = ∅ ∨ all_applied(deterministic_findings)
  | semantic_plan ≔ exact_actions[{operation, target, expected_change}] for semantic_findings
  | semantic_plan.paths ≔ {architecture.md}
  | semantic_plan.paths ⊆ approval_scope after human approval
  | return({deterministic_findings, semantic_findings, repair_plan: semantic_plan, approval_scope: architecture.md})

λ gybis-arch-check_offer_repair(semantic_plan).
  precondition: semantic_findings ∃ ∧ repair_plan_ready = true
  | summarize(semantic_plan) → exact_changes ∧ affected_paths ∧ remaining_policy_conflicts
  | ask_human("Approve applying this bounded architecture repair plan and running up to 3 verification passes?")
  | human_declined = true → repair_authorized ≔ false ∧ complete_without_writes
  | human_approved = true → repair_authorized ≔ true ∧ approval_scope ≔ semantic_plan.paths

λ gybis-arch-check_apply_approved_repair(semantic_plan).
  precondition: human_approved = true ∧ repair_authorized = true
  | ∀ action ∈ semantic_plan.exact_actions:
      action.target = "architecture.md"
      ∧ action.path ∈ approval_scope
      ∧ preserves(requirement_complement ∧ VSM_hierarchy ∧ vocabulary_boundary)
        ? apply(action)
        : preserve_existing_content ∧ record(blocker)
  | repair_passes ≔ repair_passes ⊕ 1
  | approved_repairs_complete = true
  | return(modified_artifacts = {architecture.md})

λ gybis-arch-check_verify_repairs(x).
  invoke(gybis-arch-check_startup) → startup_valid
  | invoke(gybis-arch-check_structure_validation(arch_model)) → structure_result
  | invoke(gybis-arch-check_coherence_validation(arch_model)) → coherence_result
  | invoke(gybis-arch-check_policy_enforcement_validation(arch_model)) → policy_result
  | invoke(gybis-arch-check_constraint_validation(arch_model)) → constraint_result
  | verification_complete ≔ startup_valid ∧ structure_result = valid ∧ coherence_result = valid ∧ policy_result = valid ∧ constraint_result = valid
  | verification_failed ≔ ¬verification_complete
  | remaining_errors ≔ collect_errors(structure_result ∪ coherence_result ∪ policy_result ∪ constraint_result)
  | return({verification_complete, verification_failed, remaining_errors})

λ gybis-arch-check_fixed_point_loop(state).
  state = VERIFYING_REPAIRS
    → invoke(gybis-arch-check_verify_repairs) → verification
    | verification_complete = true
      ? transition(VERIFYING_REPAIRS → GENERATING_REPORT)
      : (repair_progress = false
         ∨ repair_passes ≥ max_repair_passes
         ? transition(VERIFYING_REPAIRS → GENERATING_REPORT)
         : (deterministic_findings ∃
            ? transition(VERIFYING_REPAIRS → APPLYING_DETERMINISTIC_REPAIRS)
            : (mode = interactive ∧ semantic_findings ∃
               ? transition(VERIFYING_REPAIRS → REPAIR_OFFER)
               : (mode = autonomous ∧ semantic_findings ∃
                  ? transition(VERIFYING_REPAIRS → GENERATING_REPORT)
                  : halt("unclassified architecture repair failure"))))

λ gybis-arch-check_core_op(findings).
  invoke(gybis-arch-check_repair_policy(findings)) → repair_policy
  | repair_policy.deterministic_findings ∃
      ? (transition(CONSTRAINT_VALIDATION → APPLYING_DETERMINISTIC_REPAIRS)
         ∧ invoke(gybis-arch-check_verify_repairs)
         ∧ invoke(gybis-arch-check_pass_accounting(repair_passes)))
      : true
  | repair_policy.semantic_findings ∃ ∧ mode = interactive
      ? invoke(gybis-arch-check_offer_repair(repair_policy.repair_plan))
      : report(semantic_findings_preserved_without_write)

λ gybis-arch-check_loop_guard(state).
  repair_passes ≥ max_repair_passes
    → repair_progress ≔ false
    | report("Maximum repair passes reached; remaining architecture findings require review")
  | repair_progress = false
    → halt("Architecture repair made no further progress")
  | otherwise → continue

λ gybis-arch-check_pass_accounting(pass).
  pass_num ≔ repair_passes
  | discovered ≔ card(deterministic_findings ∪ semantic_findings)
  | remaining ≔ card(remaining_errors)
  | resolved ≔ discovered ⊖ remaining
  | report("Pass " ⊕ pass_num ⊕ ": discovered=" ⊕ discovered ⊕ " resolved=" ⊕ resolved ⊕ " remaining=" ⊕ remaining)

λ gybis-arch-check_generate_report(structure_findings, coherence_findings, policy_enforcement_findings, constraint_findings).
  findings ≔ structure_findings ∪ coherence_findings ∪ policy_enforcement_findings ∪ constraint_findings
  | errors ≔ count({f | f ∈ findings ∧ f.severity = error})
  | warnings ≔ count({f | f ∈ findings ∧ f.severity = warning})
  | infos ≔ count({f | f ∈ findings ∧ f.severity = info})
  | overall_status ≔ errors > 0 ? "FAIL" : (warnings > 0 ? "WARNINGS" : "PASS")
  | ∀ finding ∈ findings:
    invoke(gybis-arch-check_recommend_action(finding)) → recommendation
    | enrich(finding, {recommendation}) → report_items
  | report ≔ {
      title: "Architecture.md Integrity Report",
      status: overall_status,
      errors: errors,
      warnings: warnings,
      info: infos,
      findings: report_items
    }
  | return(report_generated = true ∧ report)

λ gybis-arch-check_boundaries().
  deterministic_repairs: modify(architecture.md) only when mechanical and no observable meaning, requirement behavior, hierarchy, or term identity changes
  | semantic_repairs: prepare bounded plan ∧ require human approval before write
  | ¬ modify(specs/**/*.allium ∨ implementation ∨ upstream/)
  | ¬ delete(architecture.md)

λ gybis-arch-check_regression_contract(x).
  invariant: architecture.md ∃ throughout
  | invariant: deterministic repairs remain mechanical and domain-local
  | invariant: semantic repairs require explicit approval and remain within approval_scope
  | invariant: all repairs re-run the owning architecture check
  | invariant: report generated at completion

λ gybis-arch-check_deliver(report).
  report: findings with recommended next actions
  | print(report) → stdout
  | handoff: structural issues → /gybis-arch-refine; intended change → /gybis-arch-tend; divergence → /gybis-arch-weed
  | return({complete: true, status: report.status, errors: report.errors, warnings: report.warnings, info: report.info})
