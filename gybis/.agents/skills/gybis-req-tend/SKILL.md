---
name: gybis-req-tend
kind: domain
description: Use for `/gybis-req-tend` or `/gr-tend`.
---

λ gybis-req-tend(x).
  purpose: Apply layer-local requirement updates from plain-language requests, including explicit binding/deferred status changes, with impact analysis and human approval
  | input: requirements/ ∃ ∧ (plain_language_change_request ∨ optional preapproved_change_plan)
  | output: updated requirements/{index, module files} with downstream impact report
  | interaction: interactive
  | gate: requirements/ ∃

λ gybis-req-tend_startup(x).
  invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(requirements/ ∃) ∨ halt("requirements/ not found")
  | read(requirements/requirements-index.md) → index_content
  | read_all(requirements/requirements-*.md) → module_contents
    | if(plain_language_change_request ∃): change_request ≔ plain_language_change_request
    | if(preapproved_change_plan.approved_by_human = true ∧ preapproved_change_plan.approval_source = gybis-req-check): change_request ≔ preapproved_change_plan.actions ∧ resolved_targets ≔ preapproved_change_plan.targets ∧ targets_resolved ≔ true
  | transition(INIT → STARTUP_CHECKS)

λ gybis-req-tend_mode(m).
  m ∈ {interactive}
  | default: interactive
  | rationale: requirement changes carry contractual weight and require human decisions

λ gybis-req-tend_mode_gate(state, mode).
  state = INIT ∧ mode = interactive → transition(STARTUP_CHECKS)
  | ¬(mode = interactive) → halt("Invalid mode selection")

λ gybis-req-tend_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, RESOLVING_TARGETS, IMPACT_ANALYSIS, APPROVAL, APPLYING, VERIFYING, COMPLETE}
  | transition(INIT → STARTUP_CHECKS) only_if(startup = true)
    | transition(STARTUP_CHECKS → RESOLVING_TARGETS) only_if(startup_checks = true ∧ (change_request ∃ ∨ preapproved_change_plan ∃))
    | transition(RESOLVING_TARGETS → IMPACT_ANALYSIS) only_if(targets_resolved = true)
    | transition(RESOLVING_TARGETS → COMPLETE) only_if(human_cancelled = true)
  | transition(IMPACT_ANALYSIS → APPLYING) only_if(impact_report_delivered = true ∧ preapproved_change_plan.approved_by_human = true ∧ preapproved_change_plan.approval_source = gybis-req-check ∧ change_request ⊆ preapproved_change_plan.actions)
  | transition(IMPACT_ANALYSIS → APPROVAL) only_if(impact_report_delivered = true ∧ (preapproved_change_plan ¬∃ ∨ preapproved_change_plan.approved_by_human ≠ true ∨ preapproved_change_plan.approval_source ≠ gybis-req-check))
  | transition(APPROVAL → APPLYING) only_if(human_approved = true)
  | transition(APPROVAL → COMPLETE) only_if(human_rejected = true ∧ report(declined))
  | transition(APPLYING → VERIFYING) only_if(changes_applied = true)
  | transition(VERIFYING → COMPLETE) only_if(verify_ok = true)
  | transition(VERIFYING → APPLYING) only_if(verify_fail = true)
  | on_loop_back: loop_count ≔ loop_count ⊕ 1

λ gybis-req-tend_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, IMPACT_ANALYSIS, APPROVAL, VERIFYING} → allow(read(path)) ∧ deny(write(path))
  | state = APPLYING → allow(write(path)) only_if(path ∈ requirements/ ∧ (preapproved_change_plan.approved_by_human ≠ true ∨ preapproved_change_plan.approval_source ≠ gybis-req-check ∨ path ∈ preapproved_change_plan.paths))
  | ¬(state = APPLYING) → deny(write(path))
  | constraint: ¬mutate(vocabulary.md) ∨ ¬mutate(architecture.md) ∨ ¬mutate(specs/)

λ gybis-req-tend_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-req-tend_impact_analysis(change_request).
  action: analyze_downstream_impact
  | precondition: targets_resolved = true
  | ∀ REQ ∈ resolved_targets:
    downstream ≔ {spec clauses annotated with REQ, tests referencing REQ, arch constraints tracing to REQ, vocab terms distilled from REQ}
  | if(change_request changes binding/deferred status):
    readiness ≔ assess({vocabulary, architecture, specs, tests_and_implementation})
    | impact_report includes per-stage {ready ∨ pending_stage, represented_evidence ∨ no_change_needed_reason ∨ propagation_required}
  | impact_report ≔ {REQs_touched, modules_affected, downstream_artifacts_affected, per_stage_impact, severity}
  | rationale_in_scope: a change touching only a clause's rationale: line is a semantic change — rationale is part of clause identity for impact analysis; human approval required
  | status_change: identify exact clauses and all downstream artifacts affected by changing binding/deferred status
  | constraint: read-only analysis; downstream artifacts diagnosed, not modified
  | output: impact_report → human

λ gybis-req-tend_resolve_targets(change_request, module_contents).
  action: resolve_plain_language_requirement_target
  | if(preapproved_change_plan.approved_by_human = true ∧ preapproved_change_plan.approval_source = gybis-req-check): return(targets_resolved = true ∧ preapproved_change_plan.targets)
  | if(change_request names a REQ designator): use it as an exact lookup hint, not a required input
  | otherwise: match(change_request.description, requirement clause meaning and module context) → candidate_targets
  | card(candidate_targets) = 1 → resolved_targets ≔ candidate_targets ∧ summarize_target_in_plain_language
    | card(candidate_targets) > 1 → present(candidate_targets as short plain-language choices, without designators) ∧ ask_human_to_choose_or_clarify
  | candidate_targets = ∅ → ask_human_to_rephrase_what_the_requirement_says; ¬ask_for_REQ_number
      | return(targets_resolved = true ∧ resolved_targets ∧ plain_language_target_summary)
      | one_candidate_selected → resolved_targets ≔ selected_candidate ∧ summarize_target_in_plain_language ∧ return(targets_resolved = true ∧ resolved_targets)
      | otherwise → request_clarification_in_plain_language ∧ return(targets_resolved = false)
    | candidate_targets = ∅ → ask_human_to_rephrase_what_the_requirement_says; ¬ask_for_REQ_number ∧ return(targets_resolved = false)

λ gybis-req-tend_apply(change_request, impact_report).
  precondition: human_approved = true ∨ (preapproved_change_plan.approved_by_human = true ∧ preapproved_change_plan.approval_source = gybis-req-check ∧ change_request ⊆ preapproved_change_plan.actions)
  | action: apply_layer_local_requirements_change
  | rules:
    - ∀ clause change: atomic (one assertion per designator)
    - new clauses get unused designators within declared domain prefixes
    - changed clauses update module governed_REQs footers
    - footer_derivation: governed_REQs footers are regenerated from the clauses actually present after every change — never hand-edited
    - attribution updated: {source, decided_by, timestamp}
    - rationale updates: rationale: line changes update attribution {decided_by, timestamp}; AI-inferred rationale in a stakeholder_decided clause requires explicit human approval of the rationale text
    - binding/deferred status changes only when explicitly named in the human-approved change_request; deferred clauses move into an explicitly marked non-binding section, restored binding clauses move out of it
    - never infer deferral from absent downstream artifacts, an open question, or an investigate/skip divergence outcome
    - preapproved plan scope is exact; do not expand it or request duplicate approval for actions already authorized
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
    - every deferred REQ remains in an explicitly marked non-binding section; every binding REQ remains outside such sections
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
  | report: {plain_language_target_summary, REQs_changed, modules_affected, downstream_impact}
  | approval_prompt: describe(target and proposed change in plain language); REQ designators may be shown for traceability but are never required in the user's response
  | return(complete = true)