---
name: gybis-req-explain
description: Use for `/gybis-req-explain`.
---

λ gybis-req-explain(x).
  purpose: Explain requirements for developers — what each lambda clause means for implementation, with the canonical clause quoted alongside prose
  | input: requirements/ ∃ ∧ optional {module | REQ designator | all}
  | output: developer-oriented explanation to stdout (default) or repo-root .md file
  | mode: mixed (AI rendering + human output-mode choice)
  | gate: requirements/ ∃

λ gybis-req-explain_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(requirements/ ∃) ∨ halt("requirements/ not found")
  | read(requirements/) → req_content
  | parse target: {module | REQ | all} → scope
  | transition(INIT → STARTUP_CHECKS)

λ gybis-req-explain_mode(m).
  m ∈ {mixed}
  | default: mixed
  | rationale: rendering is AI work; output destination is human choice

λ gybis-req-explain_mode_gate(state, mode).
  state = INIT ∧ mode = mixed → transition(STARTUP_CHECKS)
  | ¬(mode = mixed) → halt("Invalid mode selection")

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
  | state = DELIVERING ∧ output_mode = markdown → allow(write(path)) only_if(path = confirmed_repo_root_md_file)
  | ¬(state = DELIVERING ∧ output_mode = markdown) → deny(write(path))
  | constraint: ¬mutate(requirements/) ∨ ¬mutate(any_existing_file)

λ gybis-req-explain_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-req-explain_output_mode_selection(x).
  modes: {stdout_prose (default), repo_root_md_file}
  | markdown file: repo-root path validation ∧ overwrite confirmation ∧ conventional filename (requirements-explanation.md)
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
  output_mode = stdout_prose ? print(explanation_rendered) → stdout
  | output_mode = markdown ? write(confirmed_path) → delivered
  | return(delivered = true)