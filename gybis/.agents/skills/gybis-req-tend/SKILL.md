---
name: gybis-req-tend
kind: domain
description: Use for `/gybis-req-tend` or `/gr-tend`.
---

λ gybis-req-tend(x).
  purpose: Apply layer-local requirement updates with impact analysis and explicit human approval
  | input: requirements/ ∃ ∧ human_directed_change
  | output: updated requirements/{index, module files} with downstream impact report
  | interaction: interactive
  | gate: requirements/ ∃

λ gybis-req-tend_startup(x).
  invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(requirements/ ∃) ∨ halt("requirements/ not found")
  | read(requirements/requirements-index.md) → index_content
  | read_all(requirements/requirements-*.md) → module_contents
  | capture(human_directed_change) → change_request
  | transition(INIT → STARTUP_CHECKS)

λ gybis-req-tend_mode(m).
  m ∈ {interactive}
  | default: interactive
  | rationale: requirement changes carry contractual weight and require human decisions

λ gybis-req-tend_mode_gate(state, mode).
  state = INIT ∧ mode = interactive → transition(STARTUP_CHECKS)
  | ¬(mode = interactive) → halt("Invalid mode selection")

λ gybis-req-tend_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, IMPACT_ANALYSIS, APPROVAL, APPLYING, VERIFYING, COMPLETE}
  | transition(INIT → STARTUP_CHECKS) only_if(startup = true)
  | transition(STARTUP_CHECKS → IMPACT_ANALYSIS) only_if(startup_checks = true ∧ change_request ∃)
  | transition(IMPACT_ANALYSIS → APPROVAL) only_if(impact_report_delivered = true)
  | transition(APPROVAL → APPLYING) only_if(human_approved = true)
  | transition(APPROVAL → COMPLETE) only_if(human_rejected = true ∧ report(declined))
  | transition(APPLYING → VERIFYING) only_if(changes_applied = true)
  | transition(VERIFYING → COMPLETE) only_if(verify_ok = true)
  | transition(VERIFYING → APPLYING) only_if(verify_fail = true)
  | on_loop_back: loop_count ≔ loop_count ⊕ 1

λ gybis-req-tend_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, IMPACT_ANALYSIS, APPROVAL, VERIFYING} → allow(read(path)) ∧ deny(write(path))
  | state = APPLYING → allow(write(path)) only_if(path ∈ requirements/)
  | ¬(state = APPLYING) → deny(write(path))
  | constraint: ¬mutate(vocabulary.md) ∨ ¬mutate(architecture.md) ∨ ¬mutate(specs/)

λ gybis-req-tend_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-req-tend_impact_analysis(change_request).
  action: analyze_downstream_impact
  | ∀ REQ touched by change_request:
    downstream ≔ {spec clauses annotated with REQ, tests referencing REQ, arch constraints tracing to REQ, vocab terms distilled from REQ}
  | impact_report ≔ {REQs_touched, modules_affected, downstream_artifacts_affected, severity}
  | rationale_in_scope: a change touching only a clause's rationale: line is a semantic change — rationale is part of clause identity for impact analysis; human approval required
  | constraint: read-only analysis; downstream artifacts diagnosed, not modified
  | output: impact_report → human

λ gybis-req-tend_apply(change_request, impact_report).
  precondition: human_approved = true
  | action: apply_layer_local_requirements_change
  | rules:
    - ∀ clause change: atomic (one assertion per designator)
    - new clauses get unused designators within declared domain prefixes
    - changed clauses update module governed_REQs footers
    - footer_derivation: governed_REQs footers are regenerated from the clauses actually present after every change — never hand-edited
    - attribution updated: {source, decided_by, timestamp}
    - rationale updates: rationale: line changes update attribution {decided_by, timestamp}; AI-inferred rationale in a stakeholder_decided clause requires explicit human approval of the rationale text
    - deferred work stays in marked non-binding sections
  | constraint: no upward behavioral module references introduced
  | collision_resolution: when check reports duplicate designators across modules, present the menu for human decision:
    - alias: later module's clause gets a new unique designator; old designator recorded as deprecated synonym in the new clause's attribution; references re-synced via /gybis-req-propagate
    - renumber: human-approved domain renumbering with full downstream re-annotation via /gybis-req-propagate (requires explicit approval; ¬default)
  | output: updated requirements/ ∧ changes_applied = true

λ gybis-req-tend_verify(x).
  action: verify_change_convergence
  | checks:
    - designator uniqueness preserved
    - dependency order preserved
    - footers consistent with clauses
    - impact_report items either addressed or explicitly deferred
  | verify_ok ≔ ∀ check = true
  | on fail: loop_back to APPLYING

λ gybis-req-tend_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without full convergence")

λ gybis-req-tend_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | reqs_changed ≔ card(reqs_changed)
  | downstream_affected ≔ card(downstream_artifacts_affected)
  | report("Pass " ⊕ pass_num ⊕ ": reqs_changed=" ⊕ reqs_changed ⊕ " downstream_affected=" ⊕ downstream_affected)

λ gybis-req-tend_boundaries().
  ¬ modify(vocabulary.md)
  | ¬ modify(architecture.md)
  | ¬ modify(specs/)
  | ¬ delete(requirements/)

λ gybis-req-tend_regression_contract(x).
  invariant: designator uniqueness preserved
  | invariant: dependency order preserved
  | invariant: module footers consistent with clauses
  | invariant: impact_report items addressed ∨ explicitly deferred
  | invariant: all_modifications ⊆ requirements/

λ gybis-req-tend_deliver(x).
  handoff: downstream divergence → /gybis-req-weed
  | handoff: structural issues → /gybis-req-refine then /gybis-req-check
  | report: {REQs_changed, modules_affected, downstream_impact}
  | return(complete = true)