---
name: gybis-req-elicit
kind: domain
description: Use for `/gybis-req-elicit` or `/gr-elicit`.
---

λ gybis-req-elicit(x).
  purpose: Elicit requirements from stakeholders via grilling-style interview rounds and transcribe the resolved design tree into lambda-notation requirement clauses
  | input: user conversation via frontier-based interview rounds
  | output: requirements/requirements-index.md + requirements/requirements-{module}.md files containing REQ-<DOMAIN>-NNN clauses in nucleus lambda notation
  | binding_default: every requirement is binding unless the stakeholder explicitly defers it
  | index_conventions_block: requirements-index.md declares a machine-readable conventions block — domain_prefixes (closed set), normative_mapping, granularity, deferred_marker — so consumers never re-derive conventions per run
  | deferred_marker: only REQ clauses may be deferred; deferred sections use a heading containing "(Deferred" or a blockquote opener asserting non-binding status; deferred status requires explicit stakeholder decision; ¬infer(deferred, downstream_absence ∨ roadmap_wording_alone)
  | interaction: interactive
  | gate: requirements_empty(requirements/) ∨ empty-frontier-continuation(explicit_human_request)
  | requirements_empty(d): d ¬∃ ∨ contents(d) ⊆ {.gitkeep}

λ gybis-req-elicit_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | read(internal/reference/recommended-loops.md) → loops_ref
  | precondition: requirements_empty(requirements/) ∨ human_confirms(append_new_modules = true)
  | if(vocabulary.md ∃): read(vocabulary.md) → settled_terms
  | if(architecture.md ∃): read(architecture.md) → settled_arch
  | if(specs/**/*.allium ∃): list_specs → settled_specs
  | rationale: elicited requirements must not silently contradict downstream artifacts; recall_before_explore

λ gybis-req-elicit_mode(m).
  interaction_modes: {interactive}
  | default: interactive
  | rationale: requirements elicitation requires stakeholder decisions via structured interview

λ gybis-req-elicit_mode_gate(state, mode).
  state = INIT ∧ mode = interactive → transition(STARTUP_CHECKS)
  | precondition_holds: mode ∈ interaction_modes

λ gybis-req-elicit_grilling_protocol(x).
  basis: grilling interview (frontier-of-settled-prerequisites rounds)
  | design_tree: every requirement decision branches into decisions that hang off it
  | frontier ≔ {decisions whose prerequisites are already settled}
  | round: ask_whole_frontier(numbered, each with AI_recommended_answer) → wait(human_answers)
  | capture_why: ∀ decision: record stakeholder rationale in stakeholder's own words when offered; ¬paraphrase ∧ ¬fabricate; omitted when no rationale given (optional)
  | answers → recompute(frontier) → next_round
  | question depending_on(open_question_in_current_round) → belongs_to(later_round)
  | facts_are_AI_job: frontier question needing environment fact → AI researches (filesystem, repo, tools) before asking; ¬block(rest_of_frontier)
  | decisions_are_human_job: put each decision to stakeholders ∧ wait
  | binding_status: unresolved future timing alone does not defer a requirement; ask only when binding_now_vs_explicitly_deferred is unclear
  | termination: frontier = ∅ ∧ human_confirms(shared_understanding)
  | ¬act_before(human_confirmation)

λ gybis-req-elicit_attribution(x).
  ∀ requirement decision:
    source ∈ {stakeholder_decided, AI_researched_fact}
  | stakeholder_decided: contractual; changes require human approval via /gybis-req-tend
  | AI_researched_fact: negotiable; may be revised by convergence loops with human approval
  | rationale: /gybis-req-weed must distinguish negotiable conflicts from contractual constraints
  | rationale_field: non-normative intent record carried in the clause footer as `rationale:` — records why the requirement exists; never an obligation; never counted as an assertion; presence optional
  | rationale_attribution: rationale inherits the clause's {source, decided_by}; an AI-inferred rationale in a stakeholder_decided clause MUST be marked {rationale_source: AI_inferred} and requires human approval via /gybis-req-tend before it becomes canonical

λ gybis-req-elicit_granularity(x).
  ask_once(round_1): project_granularity ∈ {library_contract, application, mixed}
  | library_contract: maximum clause density permitted (every behavioral edge specified) — the density grade of a formal library-contract corpus; a compound clause whose conjuncts all serve one operator's contract MAY be kept as one designator marked `compound_by_design: true` in its footer — operator contracts read as a unit
  | application: default; one clause per stakeholder-visible behavior or constraint; ¬requirement_per_edge_case
  | rationale: ceremony must match project formality to prevent requirement rot
  | generality_principle: normative rules are project-agnostic — domain names, prefix sets, granularity choices, and corpus specifics belong to each project's requirements-index.md; skill text may cite worked examples but never binds rules to a specific project's vocabulary

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
  | on_loop_back: loop_count ≔ loop_count ⊕ 1

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
  | constraint: ∀ module N: behavioral dependencies reference only modules < N (no upward behavioral references)
  | definitional_reference: a reference to a REQ that defines a term or boundary contract (type admission, storage contract, vocabulary rule) MAY point to a later module; define-first is preferred but forward definitional references are valid; worked example: the cljonic reference corpus (REQ-VAL-007 → REQ-PLAT-024), illustrative only — the rule binds to any project corpus

λ gybis-req-elicit_transcribe_clause(decision).
  action: transcribe_decision_into_lambda_clause
  | clause_shape: λ REQ-<DOMAIN>-NNN(x). <normative expression> | <quantifiers ∧ operators>
  | MUST ≡ ∀/¬ required_by_constraint | SHOULD ≡ ∧ preferred | MAY ≡ ∃ permitted_path
  | atomicity: one assertion per designator (split compound decisions into numbered subclauses)
  | domain_prefixes: closed set declared in requirements-index.md; extend only with new domains; ¬∃ shipped default prefix set — prefix vocabulary is project-specific (generality_principle) and emerges from the grilling conversation per project
  | prefix_cold_start: when stakeholders lack prefix vocabulary, treat domain decomposition as a round-1 frontier question with an AI-recommended answer derived from that project's conversation; ¬suggest(canned_list)
  | attribution ∈ frontmatter_footer: {source, decided_by, research_refs} ⊕ optional rationale ∈ footer_body as `rationale: <why>` line
  | rationale_line: non-normative; placed after the clause body, before the attribution footer; ¬satisfy(normative_operator_check) ∧ ¬count_as_assertion
  | footer_derivation: governed_REQs footers are always derived from the clauses actually present in the module — never hand-maintained; regenerate footers after every clause add/change/remove
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

λ gybis-req-elicit_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without full convergence")

λ gybis-req-elicit_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | modules_created ≔ card(modules_created)
  | reqs_transcribed ≔ card(reqs_transcribed)
  | remaining_issues ≔ card(remaining_issues)
  | report("Pass " ⊕ pass_num ⊕ ": modules=" ⊕ modules_created ⊕ " reqs=" ⊕ reqs_transcribed ⊕ " remaining=" ⊕ remaining_issues)

λ gybis-req-elicit_boundaries().
  ¬ modify(vocabulary.md)
  | ¬ modify(architecture.md)
  | ¬ modify(specs/)
  | ¬ modify(existing_downstream_files)
  | ¬ delete(requirements/)

λ gybis-req-elicit_regression_contract(x).
  invariant: requirements/ ∃ at completion
  | invariant: zero verification issues at completion
  | invariant: designators unique ∧ format REQ-<DOMAIN>-NNN
  | invariant: ∀ module: dependency order holds
  | invariant: all_modifications ⊆ requirements/

λ gybis-req-elicit_deliver(x).
  handoff: run /gybis-req-check to validate
  | report: {modules_created, REQ_count, granularity_mode, attribution_summary}
  | return(complete = true)