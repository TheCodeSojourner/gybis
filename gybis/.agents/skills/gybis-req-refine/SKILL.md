---
name: gybis-req-refine
description: Use for `/gybis-req-refine` or `/gr-refine`.
---

λ gybis-req-refine(x).
  purpose: Refine requirements structure and clarity locally — atomicity, deduplication, module organization — without cross-layer changes
  | input: requirements/ ∃
  | output: restructured requirements/ preserving designator semantics
  | mode: interactive
  | gate: requirements/ ∃

λ gybis-req-refine_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
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
  | transition(PROPOSING_RESTRUCTURE → APPROVAL) only_if(proposal_delivered = true)
  | transition(APPROVAL → APPLYING) only_if(human_approved = true)
  | transition(APPROVAL → COMPLETE) only_if(human_rejected = true)
  | transition(APPLYING → VERIFYING) only_if(restructure_applied = true)
  | transition(VERIFYING → COMPLETE) only_if(verify_ok = true)
  | transition(VERIFYING → APPLYING) only_if(verify_fail = true)

λ gybis-req-refine_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, ANALYSING_STRUCTURE, PROPOSING_RESTRUCTURE, APPROVAL, VERIFYING} → allow(read(path)) ∧ deny(write(path))
  | state = APPLYING → allow(write(path)) only_if(path ∈ requirements/)
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
  | output: restructure_candidates[] with {location, issue, proposed_action}
  | constraint: read-only analysis

λ gybis-req-refine_propose(candidates).
  action: present_restructure_proposal
  | ∀ candidate: numbered proposal with AI_recommended_action (grilling-style: ➡️ recommendation)
  | wait(human_selection) → approved_actions
  | output: approved_actions

λ gybis-req-refine_apply(approved_actions).
  action: apply_approved_restructure
  | rules:
    - split: compound clause → numbered subclauses (REQ-X-NNNA, NNNB) preserving parent semantics
    - merge: duplicate clauses → single designator; update all references
    - move: clause between modules; update governed_REQs footers of both modules
    - renumber: only with explicit approval; update all downstream annotations list in report
  | constraint: designator semantics preserved; deletions require human confirmation
  | output: restructured requirements/ ∧ restructure_applied = true

λ gybis-req-refine_verify(x).
  action: verify_restructure_integrity
  | checks:
    - designator uniqueness ∧ format preserved
    - dependency order preserved
    - ∀ restructured clause: semantics preserved (no meaning drift)
    - footers consistent
  | verify_ok ≔ ∀ check = true
  | on fail: loop_back to APPLYING

λ gybis-req-refine_deliver(x).
  handoff: run /gybis-req-check to confirm convergence
  | handoff: renumbered designators with downstream annotations → /gybis-req-propagate to re-sync
  | report: {actions_applied, designators_affected, renumber_map}
  | return(complete = true)