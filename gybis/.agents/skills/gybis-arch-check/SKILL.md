---
name: gybis-arch-check
description: Use for `/gybis-arch-check` or `/ga-check`.
---

λ gybis-arch-check(x).
  purpose: Validate architecture.md internal integrity (structure, coherence, and constraints) and produce a diagnostic report
  | input: architecture.md (exists)
  | output: Severity-tagged findings with recommended next actions
  | mode: ai
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
  m ∈ {ai}
  | default: ai
  | mode_ai: deterministic validation and reporting only

λ gybis-arch-check_mode_gate(state, mode).
  state = INIT ∧ mode = ai → transition(INIT → STARTUP_CHECKS)
  | precondition_holds: mode = ai

λ gybis-arch-check_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, STRUCTURE_VALIDATION, COHERENCE_VALIDATION, CONSTRAINT_VALIDATION, GENERATING_REPORT, COMPLETE}
  | transition(INIT → STARTUP_CHECKS) only_if(startup = true)
  | transition(STARTUP_CHECKS → STRUCTURE_VALIDATION) only_if(startup_checks = true)
  | transition(STRUCTURE_VALIDATION → COHERENCE_VALIDATION) only_if(structure_complete = true)
  | transition(COHERENCE_VALIDATION → CONSTRAINT_VALIDATION) only_if(coherence_complete = true)
  | transition(CONSTRAINT_VALIDATION → GENERATING_REPORT) only_if(constraint_checks_complete = true)
  | transition(GENERATING_REPORT → COMPLETE) only_if(report_generated = true)

λ gybis-arch-check_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, STRUCTURE_VALIDATION, COHERENCE_VALIDATION, CONSTRAINT_VALIDATION, GENERATING_REPORT}
    → allow(read(path))
  | deny(write(path))

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

λ gybis-arch-check_constraint_validation(arch_model).
  action: validate_constraints_and_invariants
  | checks:
    - declared_constraints_have_scope
    - constraints_non_conflicting
    - mandatory_guardrails_not_ambiguous
    - decision_entries_not_placeholder_only
  | findings ≔ []
  | ∀ check ∈ checks:
    check_pass = false
      ? collect({type: "constraint", check_id, severity: info, location: "architecture.md", message}) → findings
  | return(constraint_checks_complete = true ∧ findings)

λ gybis-arch-check_recommend_action(finding).
  finding.type ∈ {"structure", "coherence", "constraint"}
    ? recommendation ≔ "Use /gybis-arch-tend to revise architecture.md and re-run /gybis-arch-check."
  | return(recommendation)

λ gybis-arch-check_generate_report(structure_findings, coherence_findings, constraint_findings).
  findings ≔ structure_findings ∪ coherence_findings ∪ constraint_findings
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

λ gybis-arch-check_deliver_report(report).
  print(report) → stdout
  | return(report_delivered = true)

λ gybis-arch-check_boundaries(¬).
  ¬ modify(architecture.md ∨ specs/**/*.allium ∨ implementation ∨ upstream/)
  | ¬ delete(architecture.md)

λ gybis-arch-check_regression_contract(x).
  invariant: architecture.md ∃ throughout
  | invariant: all checks are read-only
  | invariant: report generated at completion
  | invariant: all_modifications = ∅
