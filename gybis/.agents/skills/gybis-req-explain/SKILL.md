---
name: gybis-req-explain
kind: domain
description: Use for `/gybis-req-explain`.
---

λ gybis-req-explain(x).
  purpose: Explain requirements for developers — what each lambda clause means for implementation, with the canonical clause quoted alongside prose
  | input: requirements/ ∃ ∧ optional {module | REQ designator | all}
  | output: developer-oriented explanation to stdout (default) or repo-root .md file
  | interaction: autonomous | output_mode: human-selected
  | gate: requirements/ ∃ | explicit_human_output_selection() ≡ true

λ gybis-req-explain_startup(x).
  invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(requirements/ ∃) ∨ halt("requirements/ not found")
  | read(requirements/) → req_content
  | parse target: {module | REQ | all} → scope
  | transition(INIT → STARTUP_CHECKS)

λ gybis-req-explain_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, OUTPUT_MODE_SELECTION, READING, RENDERING, DELIVERING, COMPLETE}
  | transition(INIT → STARTUP_CHECKS) only_if(startup = true)
  | transition(STARTUP_CHECKS → OUTPUT_MODE_SELECTION) only_if(startup_checks = true)
  | transition(OUTPUT_MODE_SELECTION → READING) only_if(output_mode ∃)
  | transition(READING → RENDERING) only_if(content_read = true)
  | transition(RENDERING → DELIVERING) only_if(explanation_rendered = true)
  | transition(DELIVERING → COMPLETE) only_if(delivered = true)

λ gybis-req-explain_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, OUTPUT_MODE_SELECTION, READING, RENDERING} → allow(read(path)) ∧ deny(write(path))
  | state = DELIVERING ∧ output_mode ∈ {prompted_file_only, default_file_only} → allow(write(path)) only_if(path = confirmed_repo_root_md_file)
  | ¬(state = DELIVERING ∧ output_mode ∈ {prompted_file_only, default_file_only}) → deny(write(path))
  | constraint: ¬mutate(requirements/) ∨ ¬mutate(any_existing_file)

λ gybis-req-explain_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-req-explain_output_mode_selection(x).
  output_modes: {response_only (default), prompted_file_only, default_file_only}
  | file output: repo-root path validation ∧ overwrite confirmation
    ∧ prompted_file_only requires a requested repo-root markdown filename
    ∧ default_file_only writes conventional filename (requirements-explanation.md)
  | rationale: describe-explain output-mode convention (session-16 pattern)

λ gybis-req-explain_render(req_content, scope).
  action: render_developer_explanation
  | ∀ REQ in scope:
    - quote canonical lambda clause verbatim
    - explain implementation implications (what code/tests must satisfy it)
    - explain negotiation status: contractual (stakeholder_decided) vs negotiable (AI_researched_fact)
    - cite downstream targets: annotated spec clauses ∧ tests (when propagated)
    - when a rationale: line exists, quote it verbatim and use it to ground implementation implications; when absent, derive implications from the clause body only ∧ note "no recorded rationale"
  | structure: by module, dependency order
  | output: explanation_rendered

λ gybis-req-explain_deliver(explanation_rendered, output_mode).
  output_mode = response_only ? print(explanation_rendered) → stdout
  | output_mode ∈ {prompted_file_only, default_file_only} ? write(confirmed_path) → delivered
  | return(delivered = true)

λ gybis-req-explain_boundary().
  ¬modify(requirements/) ∧ ¬modify(any_existing_file)
  | writes_limited_to(repo_root_markdown_filename)