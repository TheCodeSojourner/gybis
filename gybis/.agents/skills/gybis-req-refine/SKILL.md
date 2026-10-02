---
name: gybis-req-refine
kind: domain
description: Use for `/gybis-req-refine` or `/gr-refine`.
---

λ gybis-req-refine(x).
  purpose: Refine requirements structure and clarity locally — atomicity, deduplication, module organization — while preserving binding/deferred scope and avoiding cross-layer changes
  | input: requirements/ ∃ ∧ optional preapproved_repair_plan
  | output: restructured requirements/ preserving designator semantics
  | interaction: interactive
  | gate: requirements/ ∃

λ gybis-req-refine_startup(x).
  invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(requirements/ ∃) ∨ halt("requirements/ not found")
  | read(requirements/requirements-index.md) → index_content
  | read_all(requirements/requirements-*.md) → module_contents
  | transition(INIT → STARTUP_CHECKS)

λ gybis-req-refine_mode(m).
  m ∈ {interactive}
  | default: interactive
  | rationale: restructuring decisions (split, merge, renumber) get human confirmation

λ gybis-req-refine_mode_gate(state, mode).
  state = INIT ∧ mode = interactive → transition(STARTUP_CHECKS)
  | ¬(mode = interactive) → halt("Invalid mode selection")

λ gybis-req-refine_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, ANALYSING_STRUCTURE, PROPOSING_RESTRUCTURE, APPROVAL, APPLYING, VERIFYING, COMPLETE}
  | transition(INIT → STARTUP_CHECKS) only_if(startup = true)
  | transition(STARTUP_CHECKS → ANALYSING_STRUCTURE) only_if(startup_checks = true)
  | transition(ANALYSING_STRUCTURE → PROPOSING_RESTRUCTURE) only_if(structure_analysis_complete = true)
  | transition(PROPOSING_RESTRUCTURE → APPLYING) only_if(proposal_delivered = true ∧ preapproved_repair_plan.approved_by_human = true ∧ preapproved_repair_plan.approval_source = gybis-req-check ∧ approved_actions ⊆ preapproved_repair_plan.actions ∧ approved_actions ∃)
  | transition(PROPOSING_RESTRUCTURE → COMPLETE) only_if(proposal_delivered = true ∧ preapproved_repair_plan.approved_by_human = true ∧ approved_actions = ∅)
  | transition(PROPOSING_RESTRUCTURE → APPROVAL) only_if(proposal_delivered = true ∧ (preapproved_repair_plan ¬∃ ∨ preapproved_repair_plan.approved_by_human ≠ true ∨ preapproved_repair_plan.approval_source ≠ gybis-req-check))
  | transition(APPROVAL → APPLYING) only_if(human_approved = true)
  | transition(APPROVAL → COMPLETE) only_if(human_rejected = true)
  | transition(APPLYING → VERIFYING) only_if(restructure_applied = true)
  | transition(VERIFYING → COMPLETE) only_if(verify_ok = true)
  | transition(VERIFYING → APPLYING) only_if(verify_fail = true)
  | on_loop_back: loop_count ≔ loop_count ⊕ 1

λ gybis-req-refine_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, ANALYSING_STRUCTURE, PROPOSING_RESTRUCTURE, APPROVAL, VERIFYING} → allow(read(path)) ∧ deny(write(path))
  | state = APPLYING → allow(write(path)) only_if(path ∈ requirements/ ∧ (preapproved_repair_plan.approved_by_human ≠ true ∨ preapproved_repair_plan.approval_source ≠ gybis-req-check ∨ path ∈ preapproved_repair_plan.paths))
  | ¬(state = APPLYING) → deny(write(path))
  | constraint: ¬mutate(vocabulary.md) ∨ ¬mutate(architecture.md) ∨ ¬mutate(specs/) ∨ ¬mutate(implementation)

λ gybis-req-refine_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-req-refine_analyse_structure(module_contents).
  action: analyse_requirements_structure
  | checks:
    - compound clauses (multiple assertions per designator) → split candidates
    - duplicate or near-duplicate clauses → merge candidates
    - clauses in wrong module (foundation vs domain layering) → move candidates
    - broken or stale cross-clause references
    - prose-heavy clauses missing lambda form
    - unmarked future-work content not in deferred sections
    - explicit deferred-section membership for each REQ clause
  | output: restructure_candidates[] with {location, issue, proposed_action, deferral_status}
  | constraint: read-only analysis

λ gybis-req-refine_propose(candidates).
  action: present_restructure_proposal
  | ∀ candidate: explain proposed action, affected designators/references, and preserved deferral status
  | grouped_plan: minimize technical decisions; preserve designator identity unless renumbering is required by explicit project convention
  | grouped_plan.actions ≔ map(candidates, {owner: /gybis-req-refine, target: candidate.location, operation: candidate.proposed_action, expected_change: candidate.proposed_action})
  | proposed_actions ≔ grouped_plan.actions
  | if(preapproved_repair_plan.approved_by_human = true ∧ preapproved_repair_plan.approval_source = gybis-req-check): approved_actions ≔ proposed_actions ∩ preapproved_repair_plan.actions
  | if(preapproved_repair_plan ¬∃ ∨ preapproved_repair_plan.approved_by_human ≠ true ∨ preapproved_repair_plan.approval_source ≠ gybis-req-check): summarize(grouped_plan) → ask_human("Approve this grouped structural repair plan? [yes/no]")
    | yes → approved_actions ≔ grouped_plan.actions
    | no → approved_actions ≔ ∅
  | output: approved_actions

λ gybis-req-refine_apply(approved_actions).
  action: apply_approved_restructure
  | rules:
    - split: compound clause → numbered subclauses (REQ-X-NNNA, NNNB) preserving parent semantics and the parent's binding/deferred section membership; parent rationale: line inherited by all subclauses unless subdivided by human approval
    - merge: duplicate clauses → single designator; update all references; differing rationale: lines between merged candidates block blind merge — surfaced for human decision before merging
    - move: clause between modules; preserve its binding/deferred status and update governed_REQs footers of both modules
    - renumber: only with explicit approval; update all downstream annotations list in report
    - never create, remove, or change a deferred marker; status changes belong to /gybis-req-tend
  | constraint: designator semantics preserved; deletions require human confirmation
  | output: restructured requirements/ ∧ restructure_applied = true

λ gybis-req-refine_verify(x).
  action: verify_restructure_integrity
  | checks:
    - designator uniqueness ∧ format preserved
    - dependency order preserved
    - ∀ restructured clause: semantics preserved (no meaning drift)
    - ∀ restructured clause: binding/deferred status preserved
    - footers consistent
  | verify_ok ≔ ∀ check = true
  | on fail: loop_back to APPLYING

λ gybis-req-refine_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without full convergence")

λ gybis-req-refine_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | actions_applied ≔ card(actions_applied)
  | designators_affected ≔ card(designators_affected)
  | report("Pass " ⊕ pass_num ⊕ ": actions=" ⊕ actions_applied ⊕ " designators_affected=" ⊕ designators_affected)

λ gybis-req-refine_boundaries().
  ¬ modify(vocabulary.md)
  | ¬ modify(architecture.md)
  | ¬ modify(specs/)
  | ¬ modify(implementation)
  | ¬ delete(requirements/)

λ gybis-req-refine_regression_contract(x).
  invariant: designator uniqueness ∧ format preserved
  | invariant: dependency order preserved
  | invariant: ∀ restructured clause: semantics preserved (no meaning drift)
  | invariant: module footers consistent
  | invariant: all_modifications ⊆ requirements/

λ gybis-req-refine_deliver(x).
  handoff: run /gybis-req-check to confirm convergence
  | handoff: renumbered designators with downstream annotations → /gybis-req-propagate to re-sync
  | report: {actions_applied, designators_affected, renumber_map}
  | return(complete = true)