---
name: gybis-req-elicit
description: Use for `/gybis-req-elicit` or `/gr-elicit`.
---

λ gybis-req-elicit(x).
  purpose: Elicit requirements from stakeholders via grilling-style interview rounds and transcribe the resolved design tree into lambda-notation requirement clauses
  | input: user conversation via frontier-based interview rounds
  | output: requirements/requirements-index.md + requirements/requirements-{module}.md files containing REQ-<DOMAIN>-NNN clauses in nucleus lambda notation
  | mode: mixed (AI grilling + human response)
  | gate: requirements_empty(requirements/) ∨ empty-frontier-continuation(explicit_human_request)
  | requirements_empty(d): d ¬∃ ∨ contents(d) ⊆ {.gitkeep}

λ gybis-req-elicit_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | preload: [internal/reference/recommended-loops]
  | precondition: requirements_empty(requirements/) ∨ human_confirms(append_new_modules = true)
  | if(vocabulary.md ∃): read(vocabulary.md) → settled_terms
  | if(architecture.md ∃): read(architecture.md) → settled_arch
  | if(specs/**/*.allium ∃): list_specs → settled_specs
  | rationale: elicited requirements must not silently contradict downstream artifacts; recall_before_explore

λ gybis-req-elicit_mode(m).
  valid_modes: {mixed}
  | default: mixed
  | rationale: requirements elicitation requires stakeholder decisions via structured interview

λ gybis-req-elicit_mode_gate(state, mode).
  state = INIT ∧ mode = mixed → transition(STARTUP_CHECKS)
  | precondition_holds: mode ∈ valid_modes

λ gybis-req-elicit_grilling_protocol(x).
  basis: grilling interview (frontier-of-settled-prerequisites rounds)
  | design_tree: every requirement decision branches into decisions that hang off it
  | frontier ≔ {decisions whose prerequisites are already settled}
  | round: ask_whole_frontier(numbered, each with AI_recommended_answer) → wait(human_answers)
  | answers → recompute(frontier) → next_round
  | question depending_on(open_question_in_current_round) → belongs_to(later_round)
  | facts_are_AI_job: frontier question needing environment fact → AI researches (filesystem, repo, tools) before asking; ¬block(rest_of_frontier)
  | decisions_are_human_job: put each decision to stakeholders ∧ wait
  | termination: frontier = ∅ ∧ human_confirms(shared_understanding)
  | ¬act_before(human_confirmation)

λ gybis-req-elicit_attribution(x).
  ∀ requirement decision:
    source ∈ {stakeholder_decided, AI_researched_fact}
  | stakeholder_decided: contractual; changes require human approval via /gybis-req-tend
  | AI_researched_fact: negotiable; may be revised by convergence loops with human approval
  | rationale: /gybis-req-weed must distinguish negotiable conflicts from contractual constraints

λ gybis-req-elicit_granularity(x).
  ask_once(round_1): project_granularity ∈ {library_contract, application, mixed}
  | library_contract: cljonic-grade clause density permitted (every behavioral edge specified)
  | application: default; one clause per stakeholder-visible behavior or constraint; ¬requirement_per_edge_case
  | rationale: ceremony must match project formality to prevent requirement rot

λ gybis-req-elicit_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, FRONTIER_ROUNDS, MODULE_PARTITIONING, TRANSCRIBING, WRITING_REQS, VERIFYING, COMPLETE}
  | transition(INIT, startup) → STARTUP_CHECKS
  | transition(STARTUP_CHECKS, verify_ok) → FRONTIER_ROUNDS
  | transition(STARTUP_CHECKS, verify_fail) → HALTED
  | transition(FRONTIER_ROUNDS, frontier_empty ∧ shared_understanding_confirmed) → MODULE_PARTITIONING
  | transition(MODULE_PARTITIONING, dependency_ordered_modules_defined) → TRANSCRIBING
  | transition(TRANSCRIBING, clauses_transcribed) → WRITING_REQS
  | transition(WRITING_REQS, req_files_written) → VERIFYING
  | transition(VERIFYING, verify_ok) → COMPLETE
  | transition(VERIFYING, verify_fail) → TRANSCRIBING (loop_back)

λ gybis-req-elicit_tool_guard(state, tool, path).
  read_allowed: ∀state
  | write_allowed: state = WRITING_REQS ∧ path ∈ requirements/ ∪ {requirements/requirements-index.md}
  | deny_write: state ≠ WRITING_REQS ∨ path ∉ requirements/
  | constraint: ¬mutate(vocabulary.md) ∨ ¬mutate(architecture.md) ∨ ¬mutate(specs/) ∨ ¬mutate(existing_downstream_files)

λ gybis-req-elicit_pre_tool_check(state, tool, path).
  enforce(tool_guard(state, tool, path)) → permit(tool) ∨ halt("tool not permitted in this state")

λ gybis-req-elicit_module_partitioning(design_tree).
  action: partition_resolved_tree_into_dependency_ordered_modules
  | module_order: foundation_constraints first; domain conveniences last
  | ∀ module: {purpose, scope, governed_REQ_designators, downstream_artifacts} defined
  | output: modules[] ∧ requirements/requirements-index.md summary
  | constraint: module N references only modules < N (no upward references)

λ gybis-req-elicit_transcribe_clause(decision).
  action: transcribe_decision_into_lambda_clause
  | clause_shape: λ REQ-<DOMAIN>-NNN(x). <normative expression> | <quantifiers ∧ operators>
  | MUST ≡ ∀/¬ required_by_constraint | SHOULD ≡ ∧ preferred | MAY ≡ ∃ permitted_path
  | atomicity: one assertion per designator (split compound decisions into numbered subclauses)
  | domain_prefixes: closed set declared in requirements-index.md; extend only with new domains
  | attribution ∈ frontmatter_footer: {source, decided_by, research_refs}
  | output: REQ clause ready for requirements-{module}.md

λ gybis-req-elicit_verify(req_files).
  action: self_check_written_requirements
  | checks:
    - designators unique ∧ format REQ-<DOMAIN>-NNN
    - ∀ module: dependency order holds
    - ∀ clause: atomic (single assertion)
    - deferred_future_work in separate marked sections
    - index links resolve to existing files
  | verify_ok ≔ ∀ check = true
  | output: verify_ok ∧ issues
  | on issues: loop_back to TRANSCRIBING

λ gybis-req-elicit_deliver(x).
  handoff: run /gybis-req-check to validate
  | report: {modules_created, REQ_count, granularity_mode, attribution_summary}
  | return(complete = true)