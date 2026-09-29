---
name: gybis-req-distill
description: Use for `/gybis-req-distill` or `/gr-distill`.
---

λ gybis-req-distill(x).
  purpose: Distill initial requirements from existing architecture, specs, and implementation (brownfield entry point), producing requirement clauses plus vocabulary term candidates
  | input: architecture.md ∨ specs/**/*.allium ∨ implementation (at least one ∃)
  | output: requirements/requirements-index.md + requirements/requirements-{module}.md ∧ vocabulary term candidates list (not written to vocabulary.md)
  | mode: ai
  | gate: (architecture.md ∃ ∨ specs/**/*.allium ∃ ∨ implementation ∃) ∧ requirements/ ¬∃

λ gybis-req-distill_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | preload: [internal/allium-analyse]
  | precondition: downstream artifacts ∃ ∧ requirements/ ¬∃ ∧ artifacts_readable = true
  | gate: artifacts_readable = true → proceed ∨ halt("downstream artifacts unreadable")

λ gybis-req-distill_mode(m).
  valid_modes: {ai}
  | default: ai
  | rationale: distillation is deterministic synthesis from existing artifacts, not interactive

λ gybis-req-distill_mode_gate(state, mode).
  state = INIT ∧ mode = ai → transition(STARTUP_CHECKS)
  | precondition_holds: mode ∈ valid_modes

λ gybis-req-distill_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, READING_ARTIFACTS, ANALYSING, EXTRACTING_CONSTRAINTS, CLUSTERING_MODULES, TRANSCRIBING, EXTRACTING_TERM_CANDIDATES, WRITING_REQS, VERIFYING, COMPLETE}
  | transition(INIT, startup) → STARTUP_CHECKS
  | transition(STARTUP_CHECKS, verify_ok) → READING_ARTIFACTS
  | transition(STARTUP_CHECKS, verify_fail) → HALTED
  | transition(READING_ARTIFACTS, artifacts_read) → ANALYSING
  | transition(ANALYSING, analysis_complete) → EXTRACTING_CONSTRAINTS
  | transition(EXTRACTING_CONSTRAINTS, constraints_extracted) → CLUSTERING_MODULES
  | transition(CLUSTERING_MODULES, modules_clustered) → TRANSCRIBING
  | transition(TRANSCRIBING, clauses_transcribed) → EXTRACTING_TERM_CANDIDATES
  | transition(EXTRACTING_TERM_CANDIDATES, candidates_extracted) → WRITING_REQS
  | transition(WRITING_REQS, req_files_written) → VERIFYING
  | transition(VERIFYING, verify_ok) → COMPLETE
  | transition(VERIFYING, verify_fail) → TRANSCRIBING (loop_back)

λ gybis-req-distill_tool_guard(state, tool, path).
  read_allowed: ∀state
  | write_allowed: state = WRITING_REQS ∧ path ∈ requirements/ ∪ {requirements/requirements-index.md}
  | deny_write: state ≠ WRITING_REQS ∨ path ∉ requirements/
  | constraint: ¬mutate(vocabulary.md) ∨ ¬mutate(architecture.md) ∨ ¬mutate(specs/) ∨ ¬mutate(implementation)
  | rationale: vocabulary.md is only written by vocab skills (operator-responsibility boundary)

λ gybis-req-distill_pre_tool_check(state, tool, path).
  enforce(tool_guard(state, tool, path)) → permit(tool) ∨ halt("tool not permitted in this state")

λ gybis-req-distill_read_artifacts(x).
  action: read_all_downstream_artifacts
  | if(architecture.md ∃): read(architecture.md) → arch_content
  | if(specs/**/*.allium ∃): read_all_specs → spec_content
  | if(implementation ∃): read_sampled(implementation) → impl_content
  | output: {arch_content, spec_content, impl_content}
  | constraint: read-only access

λ gybis-req-distill_extract_constraints(arch_content, spec_content, impl_content).
  action: extract_requirement_grade_constraints
  | sources:
    - arch: non-negotiable principles, declared constraints, policy boundaries
    - specs: behavioral obligations, capacity/bounds rules, failure contracts
    - impl: platform constraints, dependency rules, resource invariants (no-alloc, no-throw, threading)
  | filter: keep only stakeholder-visible or contract-grade obligations; ¬implementation detail
  | output: constraints[] with {origin_artifact, origin_span, severity}
  | rationale: library-contract style constraints (e.g., no heap, persistence semantics) are genuine requirements for contract-grade projects

λ gybis-req-distill_cluster_modules(constraints).
  action: cluster_constraints_into_dependency_ordered_modules
  | clustering: by domain prefix candidate (REQ-PLAT, REQ-VAL, REQ-FN, REQ-<DOMAIN>...)
  | ordering: foundation constraints first; domain conveniences last
  | output: modules[] each with {purpose, scope, governed_REQs, downstream_artifacts}
  | constraint: module N references only modules < N

λ gybis-req-distill_transcribe(constraints, modules).
  action: transcribe_constraints_into_lambda_clauses
  | clause_shape: λ REQ-<DOMAIN>-NNN(x). <normative expression>
  | attribution: ∀ clause: {source: AI_researched_fact, origin_artifact, origin_span}
  | rationale_field: optional `rationale:` line per clause; when present MUST be evidence-derived — either quoted/summarized from {origin_artifact, origin_span} with {rationale_source: origin_artifact} or omitted entirely; ¬fabricate inferred rationale without explicit {rationale_source: AI_inferred} marking
  | rationale: distilled requirements are evidence-derived, not stakeholder-decided; human confirmation happens via review of output
  | output: REQ clauses grouped by module

λ gybis-req-distill_extract_term_candidates(arch_content, spec_content, impl_content).
  action: extract_vocabulary_term_candidates
  | output: term_candidates[] each with {term, definition, usage_sites, source_artifacts}
  | ¬write(vocabulary.md)
  | handoff: candidates list → /gybis-vocab-tend ingests and human approves
  | rationale: vocabulary canonical file is only mutated by vocabulary skills

λ gybis-req-distill_verify(req_files, term_candidates).
  action: self_check_written_artifacts
  | checks:
    - designators unique ∧ format valid
    - dependency order holds
    - index links resolve
    - term candidates each have definition ∧ usage sites
  | verify_ok ≔ ∀ check = true
  | on issues: loop_back to TRANSCRIBING

λ gybis-req-distill_deliver(x).
  handoff: run /gybis-req-check to validate
  | handoff: deliver term_candidates to /gybis-vocab-tend
  | report: {modules_created, REQ_count, term_candidates_count, source_artifacts}
  | return(complete = true)