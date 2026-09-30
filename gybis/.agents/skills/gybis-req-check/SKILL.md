---
name: gybis-req-check
kind: domain
description: Use for `/gybis-req-check`.
---

λ gybis-req-check(x).
  purpose: Validate requirements/ internal integrity (designators, module ordering, clause well-formedness, coverage status, traceability) and produce a diagnostic report
  | input: requirements/requirements-index.md + requirements/requirements-{module}.md (exist)
  | output: Severity-tagged findings with recommended next actions
  | interaction: autonomous
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
  m ∈ {autonomous}
  | default: autonomous
  | rationale: validation is deterministic; no human choice required

λ gybis-req-check_mode_gate(state, mode).
  state = INIT ∧ mode = autonomous → transition(INIT → STARTUP_CHECKS)
  | precondition_holds: mode = autonomous

λ gybis-req-check_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, DESIGNATOR_VALIDATION, ORDERING_VALIDATION, CLAUSE_VALIDATION, COVERAGE_VALIDATION, TRACEABILITY_VALIDATION, GENERATING_REPORT, COMPLETE}
  | transition(INIT → STARTUP_CHECKS) only_if(startup = true)
  | transition(STARTUP_CHECKS → DESIGNATOR_VALIDATION) only_if(startup_checks = true)
  | transition(DESIGNATOR_VALIDATION → ORDERING_VALIDATION) only_if(designator_checks_complete = true)
  | transition(ORDERING_VALIDATION → CLAUSE_VALIDATION) only_if(ordering_checks_complete = true)
  | transition(CLAUSE_VALIDATION → COVERAGE_VALIDATION) only_if(clause_checks_complete = true)
  | transition(COVERAGE_VALIDATION → TRACEABILITY_VALIDATION) only_if(coverage_checks_complete = true)
  | transition(TRACEABILITY_VALIDATION → GENERATING_REPORT) only_if(traceability_checks_complete = true)
  | transition(GENERATING_REPORT → COMPLETE) only_if(report_generated = true)

λ gybis-req-check_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, DESIGNATOR_VALIDATION, ORDERING_VALIDATION, CLAUSE_VALIDATION, COVERAGE_VALIDATION, TRACEABILITY_VALIDATION, GENERATING_REPORT}
    → allow(read(path))
  | deny(write(path))
  | rationale: read-only diagnostics; resolution lives in tend/weed (arch-check boundary pattern)

λ gybis-req-check_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-req-check_designator_validation(req_model).
  action: validate_designator_uniqueness_and_format
  | checks:
    - ∀ clause: designator matches REQ-<DOMAIN>-NNN
    - designators globally unique across all modules — exception: a collision declared in requirements-index.md as a known-collision (designator + governing modules listed) = warning pending /gybis-req-tend resolution; undeclared collision = error
    - domain prefixes ⊆ closed_set declared in requirements-index.md
    - numbering within domain monotone (gaps reported as info)
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

λ gybis-req-check_coverage_validation(req_model, downstream).
  action: validate_requirement_coverage_status
  | downstream ≔ specs/**/*.allium ∪ tests ∪ architecture.md (if ∃)
  | ∀ REQ clause:
    coverage ∈ {spec_clause ∃, test ∃, explicit_NA, uncovered}
  | uncovered ∧ downstream ∃ → collect({type: "uncovered_REQ", severity: warning})
  | explicit_NA with rationale → ok
  | downstream ¬∃ → coverage reported as info, not error (stage not ready yet)
  | rationale: human owns stage readiness; missing downstream artifacts are absence-of-work, not failure
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
  finding.type ∈ {designator, ordering}
    ? recommendation ≔ "Use /gybis-req-refine to restructure requirements/, then re-run /gybis-req-check."
  | finding.type = clause
    ? recommendation ≔ "Use /gybis-req-refine to split compound clauses or mark deferred work."
  | finding.type = uncovered_REQ
    ? recommendation ≔ "Use /gybis-req-propagate to annotate specs/tests, or record explicit N/A."
  | finding.type = traceability
    ? recommendation ≔ "Use /gybis-req-tend to repair footers with human approval."
  | return(recommendation)

λ gybis-req-check_generate_report(designator_findings, ordering_findings, clause_findings, coverage_findings, traceability_findings).
  findings ≔ ⋃ all finding sets
  | errors ≔ count(severity = error) | warnings ≔ count(severity = warning) | infos ≔ count(severity = info)
  | overall_status ≔ errors > 0 ? "FAIL" : (warnings > 0 ? "WARNINGS" : "PASS")
  | ∀ finding: enrich({recommendation}) → report_items
  | report ≔ {title: "Requirements Integrity Report", status: overall_status, errors, warnings, info, findings: report_items}
  | return(report_generated = true ∧ report)

λ gybis-req-check_boundaries().
  ¬ modify(requirements/ ∨ vocabulary.md ∨ architecture.md ∨ specs/**/*.allium ∨ implementation ∨ upstream/)
  | ¬ delete(requirements/)

λ gybis-req-check_regression_contract(x).
  invariant: requirements/ ∃ throughout
  | invariant: all checks are read-only
  | invariant: report generated at completion
  | invariant: all_modifications = ∅

λ gybis-req-check_deliver(report).
  report: report
  | print(report) → stdout
  | handoff: structural issues → /gybis-req-refine; intended change → /gybis-req-tend; divergence → /gybis-req-weed
  | return(complete = true)