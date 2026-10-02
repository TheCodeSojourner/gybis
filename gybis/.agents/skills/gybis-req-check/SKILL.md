---
name: gybis-req-check
kind: domain
description: Use for `/gybis-req-check`.
---

λ gybis-req-check(x).
  purpose: Validate requirements/ internal integrity and, after one explicit approval, repair safely resolvable findings in a bounded check-repair-recheck loop
  | input: requirements/requirements-index.md + requirements/requirements-{module}.md (exist)
  | output: Severity-tagged findings and, when approved, a repair and verification summary
  | interaction: interactive
  | gate: requirements/ ∃ ∧ index ∃

λ gybis-req-check_startup(x).
  invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(requirements/requirements-index.md ∃) ∨ halt("requirements index not found")
  | read(requirements/requirements-index.md) → index_content
  | index_conventions: verify index declares domain_prefixes (closed set) ∧ normative_mapping; missing conventions declarations = warning (consumers would have to re-derive them)
  | read_all(requirements/requirements-*.md) → module_contents
  | parse(module_contents) → req_model ∨ halt("requirements parse failed")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-req-check_mode(m).
  m ∈ {interactive}
  | default: interactive
  | rationale: diagnosis is deterministic; applying repairs requires one explicit human authorization

λ gybis-req-check_mode_gate(state, mode).
  state = INIT ∧ mode = interactive → transition(INIT → STARTUP_CHECKS)
  | precondition_holds: mode = interactive

λ gybis-req-check_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, DESIGNATOR_VALIDATION, ORDERING_VALIDATION, CLAUSE_VALIDATION, COVERAGE_VALIDATION, TRACEABILITY_VALIDATION, GENERATING_REPORT, REPAIR_OFFER, REPAIR_PLANNING, APPLYING_REPAIRS, VERIFYING_REPAIRS, COMPLETE}
  | INIT: repair_passes ≔ 0 ∧ repair_authorized ≔ false ∧ repair_progress ≔ true
  | transition(INIT → STARTUP_CHECKS) only_if(startup = true)
  | transition(STARTUP_CHECKS → DESIGNATOR_VALIDATION) only_if(startup_checks = true)
  | transition(DESIGNATOR_VALIDATION → ORDERING_VALIDATION) only_if(designator_checks_complete = true)
  | transition(ORDERING_VALIDATION → CLAUSE_VALIDATION) only_if(ordering_checks_complete = true)
  | transition(CLAUSE_VALIDATION → COVERAGE_VALIDATION) only_if(clause_checks_complete = true)
  | transition(COVERAGE_VALIDATION → TRACEABILITY_VALIDATION) only_if(coverage_checks_complete = true)
  | transition(TRACEABILITY_VALIDATION → GENERATING_REPORT) only_if(traceability_checks_complete = true)
  | transition(GENERATING_REPORT → REPAIR_PLANNING) only_if(report_generated = true ∧ actionable_findings ∃)
  | transition(GENERATING_REPORT → COMPLETE) only_if(report_generated = true ∧ actionable_findings = ∅)
  | transition(REPAIR_PLANNING → COMPLETE) only_if(repair_plan_ready = false ∨ repair_plan.paths = ∅ ∨ repair_progress = false)
  | transition(REPAIR_PLANNING → REPAIR_OFFER) only_if(repair_plan_ready = true ∧ repair_authorized = false)
  | transition(REPAIR_PLANNING → APPLYING_REPAIRS) only_if(repair_plan_ready = true ∧ repair_authorized = true)
  | transition(REPAIR_OFFER → COMPLETE) only_if(human_declined = true)
  | transition(REPAIR_OFFER → APPLYING_REPAIRS) only_if(human_approved = true ∧ repair_authorized = true)
  | transition(APPLYING_REPAIRS → VERIFYING_REPAIRS) only_if(repair_pass_complete = true)
  | transition(VERIFYING_REPAIRS → REPAIR_PLANNING) only_if(report_status ≠ "PASS" ∧ repair_progress = true ∧ repair_passes < max_repair_passes)
  | transition(VERIFYING_REPAIRS → COMPLETE) only_if(report_status = "PASS" ∨ repair_progress = false ∨ repair_passes ≥ max_repair_passes)
  | max_repair_passes ≔ 3

λ gybis-req-check_tool_guard(state, tool, path).
  allow(read(path))
  | state = APPLYING_REPAIRS ∧ human_approved = true ∧ path ∈ repair_plan.paths ∧ path ∈ approval_scope
    → allow(write(path))
  | otherwise → deny(write(path))
  | repair_plan.paths ⊆ {requirements/, vocabulary.md, architecture.md, specs/, test_paths}
  | rationale: diagnosis stays read-only; writes require explicit approval and must be in the bounded repair plan

λ gybis-req-check_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-req-check_designator_validation(req_model).
  action: validate_designator_uniqueness_and_format
  | checks:
    - ∀ clause: designator matches REQ-<DOMAIN>-NNN
    - designators globally unique across all modules — exception: a collision declared in requirements-index.md as a known-collision (designator + governing modules listed) = warning pending /gybis-req-tend resolution; undeclared collision = error
    - domain prefixes ⊆ closed_set declared in requirements-index.md
    - numbering within domain monotone under the ordering declared by requirements-index.md (gaps reported as info)
    - explicit project ordering requirements override the generic monotonicity heuristic; when both cannot hold, preserve the project requirement and report a checker-policy mismatch as info rather than renumbering clauses
  | findings ≔ []
  | ∀ check_pass = false: collect({type: "designator", severity: error, location, message}) → findings
  | return(designator_checks_complete = true ∧ findings)

λ gybis-req-check_ordering_validation(req_model).
  action: validate_module_dependency_ordering
  | checks:
    - module order matches requirements-index.md declared order
    - ∀ module N: behavioral dependencies reference only modules < N (no upward behavioral references)
    - definitional_forward_reference: a reference to a later module's REQ defining a term or boundary contract = info (valid); ¬∃ error for definitional references — behavioral dependency on a later module = error
    - ∀ module: {purpose, scope, governed REQs} sections nonempty
    - index links resolve to existing files (broken link = error)
  | findings ≔ collected_ordering_findings
  | return(ordering_checks_complete = true ∧ findings)

λ gybis-req-check_clause_validation(req_model).
  action: validate_clause_wellformedness
  | checks:
    - ∀ clause: lambda form `λ REQ-...-NNN(x).` present
    - ∀ clause: normative operator present (∀/¬/∧ preferred/∃ permitted) — rationale: lines excluded from this check
    - atomicity: one assertion per designator (compound clause = warning) — granularity-aware: compound clause whose footer declares `compound_by_design: true` and whose conjuncts all serve one operator's contract = info; compound clause mixing unrelated assertions = warning regardless of granularity
    - quantifiers bound (∀ has domain; ¬ has scope)
    - deferred sections marked and non-binding — marker: section heading containing "(Deferred" or deferred blockquote opener
    - compound_by_design: marker must appear in the clause footer ∧ be declared by human approval via /gybis-req-tend (¬silent lint suppression)
    - rationale: lines are non-normative and optional — ¬error(absence); a rationale: line containing normative operators (∀/¬/∃) as obligations = error (rationale must record intent, not impose constraints)
  | findings ≔ collected_clause_findings (atomicity/deferred = warning ∨ info; malformed lambda = error)
  | return(clause_checks_complete = true ∧ findings)

λ gybis-req-check_normalize_stage_result(stage, artifact_present, owner_result).
  | errors ≔
    stage = vocabulary ? owner_result.errors
    : stage = architecture ? owner_result.errors
    : stage = specs ? (owner_result = true ? 0 : 1)
    : stage = tests_and_implementation ? (owner_result = true ∨ owner_result.test_suite_passes = true ∨ owner_result.passed = true ? 0 : 1)
    : 1
  | valid ≔ artifact_present ∧ errors = 0
  | return({artifact_present, valid, errors})

λ gybis-req-check_capture_stage_check(stage, validator).
  | invoke(validator) → raw_result
  | if(validator_halts ∨ validator_returns_malformed_result): raw_result ≔ {status: "FAIL", errors: 1, diagnostic: validation_failure}
  | return(raw_result)

λ gybis-req-check_assess_stage_readiness(downstream).
  action: run_read_only_owner_checks_and_normalize_results
  | stage_results ≔ {vocabulary: {artifact_present: false, valid: false, errors: unknown}, architecture: {artifact_present: false, valid: false, errors: unknown}, specs: {artifact_present: false, valid: false, errors: unknown}, tests_and_implementation: {artifact_present: false, valid: false, errors: unknown}}
  | vocabulary.md ∃ → invoke(gybis-req-check_capture_stage_check(vocabulary, /gybis-vocab-check)) → raw_vocab_result
    | stage_results.vocabulary ≔ invoke(gybis-req-check_normalize_stage_result(vocabulary, true, raw_vocab_result))
  | architecture.md ∃ → invoke(gybis-req-check_capture_stage_check(architecture, /gybis-arch-check)) → raw_arch_result
    | stage_results.architecture ≔ invoke(gybis-req-check_normalize_stage_result(architecture, true, raw_arch_result))
  | specs/**/*.allium ∃ → invoke(internal/gybis-allium-gate(specs/)) → raw_specs_result
    | stage_results.specs ≔ invoke(gybis-req-check_normalize_stage_result(specs, true, raw_specs_result))
  | test_paths ∃ ∧ downstream.test_validation_evidence ∃ → stage_results.tests_and_implementation ≔ invoke(gybis-req-check_normalize_stage_result(tests_and_implementation, true, downstream.test_validation_evidence))
  | test_paths ∃ ∧ downstream.test_validation_evidence ¬∃ → stage_results.tests_and_implementation ≔ {artifact_present: true, valid: false, errors: unknown}
  | return(stage_results)

λ gybis-req-check_coverage_validation(req_model, downstream).
  action: validate_requirement_coverage_status
  | stage_results ≔ invoke(gybis-req-check_assess_stage_readiness(downstream))
  | deferred_REQs ≔ clauses in explicitly marked non-binding sections
  | binding_REQs ≔ all REQ clauses ∖ deferred_REQs
  | downstream_stages ≔ {vocabulary, architecture, specs, tests_and_implementation}
  | stage_ready(stage) ≡ stage_results[stage].artifact_present = true ∧ stage_results[stage].valid = true
  | ∀ REQ ∈ binding_REQs, stage ∈ downstream_stages:
    ¬stage_ready(stage) → (disposition(REQ, stage) ≔ pending_stage ∧ collect({type: "pending_stage", REQ, stage, severity: info}))
  | ∀ REQ ∈ binding_REQs, stage ∈ downstream_stages where stage_ready(stage):
    evidence(stage, REQ) ∃ → disposition(REQ, stage) ≔ represented
    | no_distinct_layer_effect(stage, REQ) → disposition(REQ, stage) ≔ no_change_needed ∧ require(reason)
    | otherwise → collect({type: "uncovered_REQ", REQ, stage, severity: warning})
  | deferred REQs: exclude from downstream coverage findings and repair candidates; report count as info
  | rationale: stage absence or invalidity is pending, not failure or deferral; binding requirements may lead downstream artifacts
  | return(coverage_checks_complete = true ∧ findings)

λ gybis-req-check_traceability_validation(req_model).
  action: validate_traceability_footer_integrity
  | checks:
    - ∀ module: governed_REQs footer matches clauses actually present
    - footer_derivation: governed_REQs footers MUST be derivable from clauses present — a footer that requires hand-maintenance to be correct = warning (regenerate from clauses)
    - ∀ module: downstream_artifact references resolvable or explicitly deferred
    - ∀ clause with attribution: source ∈ {stakeholder_decided, AI_researched_fact}
    - ∀ clause with rationale: {rationale_source: origin_artifact | AI_inferred} declared when rationale present ∧ AI_inferred rationale in stakeholder_decided clause = warning (requires human approval via /gybis-req-tend)
  | findings ≔ collected_traceability_findings
  | return(traceability_checks_complete = true ∧ findings)

λ gybis-req-check_recommend_action(finding).
  finding.type ∈ {designator, ordering} → repair_owner ≔ /gybis-req-refine
  | finding.type = clause → repair_owner ≔ /gybis-req-refine
  | finding.type = uncovered_REQ ∧ finding.stage ∈ {specs, tests_and_implementation} → repair_owner ≔ /gybis-req-propagate
  | finding.type = uncovered_REQ ∧ finding.stage = vocabulary → repair_owner ≔ /gybis-req-propagate then /gybis-vocab-tend
  | finding.type = uncovered_REQ ∧ finding.stage = architecture ∧ architecture.md ¬∃ → repair_owner ≔ /gybis-vocab-propagate
  | finding.type = uncovered_REQ ∧ finding.stage = architecture ∧ architecture.md ∃ → repair_owner ≔ /gybis-arch-tend
  | finding.type = traceability → repair_owner ≔ /gybis-req-tend
  | recommendation ≔ summarize_safe_action(finding, repair_policy)
  | return({repair_owner, recommendation})

λ gybis-req-check_repair_policy(findings).
  action: build_conservative_repair_plan
  | authority: explicit project requirements and requirements-index.md > generic checker heuristics
  | designators: preserve existing identities; renumber only when an explicit project convention requires it and all affected in-repository references can be updated
  | clauses: split only clearly independent assertions; retain a cohesive single-contract clause and add compound_by_design only when the contract is unambiguous
  | rationale: retain AI-inferred rationale only when supported by source evidence; otherwise remove the non-normative rationale line, never its requirement
  | coverage: repair binding requirements only at ready stages; deferred clauses remain out of propagation
  | if(stage_not_ready): exclude that stage from repair_plan ∧ report(pending_stage)
  | human_questions: do not ask users to resolve identifier sequencing or other mechanical classification; if requirement intent genuinely cannot be inferred, preserve it and report the blocker in plain language
  | repair_plan: exact_actions[{owner, target, operation, expected_change}] ∧ paths ⊆ writable_repair_scope ∧ preserve(normative_meaning)
  | if(repair_authorized): repair_plan.paths ⊆ approval_scope
  | return(repair_plan_ready = true ∧ repair_plan)

λ gybis-req-check_offer_repair(report).
  precondition: repair_plan_ready = true
  | summarize(repair_plan by category, proposed changes, affected paths, remaining policy conflicts)
    → ask_human("Approve applying this repair plan and running up to 3 verification passes?")
  | human_approved = false → complete_without_writes
  | human_approved = true → repair_authorized ≔ true ∧ approval_scope ≔ repair_plan.paths ∧ repair_policy
    ∧ preapproved_repair_plan ≔ {approved_by_human: true, approval_source: gybis-req-check, actions: repair_plan.exact_actions, targets: distinct(map(repair_plan.exact_actions, target)), paths: approval_scope}

λ gybis-req-check_apply_repairs(repair_plan).
  precondition: human_approved = true ∧ current_pass ≤ max_repair_passes
  | action: follow(repair_owner validation and write rules) for each repair group, within repair_plan.paths
  | owner = /gybis-req-tend → pass(preapproved_change_plan ≔ preapproved_repair_plan)
  | owner ∈ {/gybis-req-propagate, /gybis-vocab-propagate} → pass(preapproved_repair_plan)
  | owner_approval_gate: satisfied_by(approved repair_plan); do_not_open_nested_per_finding_prompts
  | no_per_finding_questions: use(repair_policy) for mechanical decisions
  | if(owner_skill_has_no_preapproved_plan_contract): do_not_invoke_as_writer ∧ report(owner_skill_handoff_required)
  | unresolved_semantic_conflict → preserve_existing_content ∧ record(blocker)
  | repair_passes ≔ repair_passes ⊕ 1
  | modified_artifacts ≔ paths_written
  | return(repair_pass_complete = true ∧ modified_artifacts)

λ gybis-req-check_verify_repairs(modified_artifacts).
  action: rerun_all_req_checks ∧ verify_each_modified_artifact
  | requirements/ modified → designator_uniqueness ∧ dependency_integrity ∧ traceability_integrity
  | specs/ modified → gybis-allium-gate = true
  | test_paths modified → test_suite_passes = true
  | architecture.md modified → vsm_coherence = true
  | vocabulary.md modified → gybis-vocab-check = true
  | repair_progress ≔ findings_decreased ∨ severity_decreased (compared_with_prior_report)
  | stop: report_status = "PASS" ∨ repair_progress = false ∨ repair_passes ≥ max_repair_passes
  | return(verification_complete = true ∧ report ∧ repair_progress)

λ gybis-req-check_generate_report(designator_findings, ordering_findings, clause_findings, coverage_findings, traceability_findings).
  findings ≔ ⋃ all finding sets
  | errors ≔ count(severity = error) | warnings ≔ count(severity = warning) | infos ≔ count(severity = info)
  | overall_status ≔ errors > 0 ? "FAIL" : (warnings > 0 ? "WARNINGS" : "PASS")
  | ∀ finding: enrich({repair_owner, recommendation}) → report_items
  | report ≔ {title: "Requirements Integrity Report", status: overall_status, errors, warnings, info, findings: report_items}
  | return(report_generated = true ∧ report)

λ gybis-req-check_boundaries().
  ¬ modify(unless human_approved = true ∧ state = APPLYING_REPAIRS)
  | modify(only paths ∈ repair_plan.paths)
  | ¬ modify(implementation ∨ upstream/ ∨ skill_files)
  | ¬ delete(requirements/)

λ gybis-req-check_regression_contract(x).
  invariant: requirements/ ∃ throughout
  | invariant: all diagnosis before approval is read-only
  | invariant: no repair writes without human_approved = true
  | invariant: all repair writes ⊆ repair_plan.paths
  | invariant: all repair writes ⊆ approval_scope
  | invariant: deferred clauses are not propagated by repair
  | invariant: not-ready stages are pending, never uncovered or explicit_NA
  | invariant: every binding REQ at every ready stage has represented evidence ∨ no_change_needed reason ∨ uncovered warning
  | invariant: repair preserves normative meaning and existing designator identity unless explicit project convention requires a change
  | invariant: repair loop terminates at PASS, no safe progress, or max_repair_passes
  | invariant: report generated at completion
  | invariant: human_declined → all_modifications = ∅

λ gybis-req-check_deliver(report).
  report: concise_summary(report) ⊕ grouped_findings(report)
  | print(report) → stdout
  | if(actionable_findings ∃): build_repair_plan(read_only) → offer_repair(report)
  | if(human_approved): fixed_point_loop(check → plan → repair → verify, max_passes = 3, ask_approval_once = true)
  | print({changes_applied, validation_results, unresolved_blockers, final_status}) → stdout
  | return(complete = true)