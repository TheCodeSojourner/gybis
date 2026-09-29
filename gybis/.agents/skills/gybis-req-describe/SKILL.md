---
name: gybis-req-describe
description: Use for `/gybis-req-describe`.
---

λ gybis-req-describe(x).
  purpose: Render requirements in business/stakeholder prose — the human-readable projection of the lambda canonical form
  | input: requirements/ ∃ ∧ optional {module | REQ designator | all}
  | output: prose description to stdout (default) or repo-root .md file
  | mode: mixed (AI rendering + human output-mode choice)
  | gate: requirements/ ∃

λ gybis-req-describe_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(requirements/ ∃) ∨ halt("requirements/ not found")
  | read(requirements/) → req_content
  | parse target: {module | REQ | all} → scope
  | transition(INIT → STARTUP_CHECKS)

λ gybis-req-describe_mode(m).
  m ∈ {mixed}
  | default: mixed
  | rationale: rendering is AI work; output destination is human choice

λ gybis-req-describe_mode_gate(state, mode).
  state = INIT ∧ mode = mixed → transition(STARTUP_CHECKS)
  | ¬(mode = mixed) → halt("Invalid mode selection")

λ gybis-req-describe_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, OUTPUT_MODE_SELECTION, READING, RENDERING, DELIVERING, COMPLETE}
  | transition(INIT → STARTUP_CHECKS) only_if(startup = true)
  | transition(STARTUP_CHECKS → OUTPUT_MODE_SELECTION) only_if(startup_checks = true)
  | transition(OUTPUT_MODE_SELECTION → READING) only_if(output_mode ∃)
  | transition(READING → RENDERING) only_if(content_read = true)
  | transition(RENDERING → DELIVERING) only_if(prose_rendered = true)
  | transition(DELIVERING → COMPLETE) only_if(delivered = true)

λ gybis-req-describe_tool_guard(state, tool, path).
  state ∈ {STARTUP_CHECKS, OUTPUT_MODE_SELECTION, READING, RENDERING} → allow(read(path)) ∧ deny(write(path))
  | state = DELIVERING ∧ output_mode = markdown → allow(write(path)) only_if(path = confirmed_repo_root_md_file)
  | ¬(state = DELIVERING ∧ output_mode = markdown) → deny(write(path))
  | constraint: ¬mutate(requirements/) ∨ ¬mutate(any_existing_file)

λ gybis-req-describe_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-req-describe_output_mode_selection(x).
  modes: {stdout_prose (default), repo_root_md_file}
  | markdown file: repo-root path validation ∧ overwrite confirmation ∧ conventional filename (requirements-description.md)
  | rationale: describe-explain output-mode convention (session-16 pattern)

λ gybis-req-describe_render(req_content, scope).
  action: render_stakeholder_prose
  | ∀ REQ in scope: lambda clause → business prose (no notation symbols, no jargon)
  | structure: by module, in dependency order, with requirement rationale
  | attribution surfaced in prose: "decided by stakeholders" vs "derived from analysis"
  | deferred sections labeled "planned future requirements (not currently binding)"
  | output: prose_rendered

λ gybis-req-describe_deliver(prose_rendered, output_mode).
  output_mode = stdout_prose ? print(prose_rendered) → stdout
  | output_mode = markdown ? write(confirmed_path) → delivered
  | return(delivered = true)