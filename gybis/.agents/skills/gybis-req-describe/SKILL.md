---
name: gybis-req-describe
kind: domain
description: Use for `/gybis-req-describe`.
---

λ gybis-req-describe(x).
  purpose: Render requirements in business/stakeholder prose — the human-readable projection of the lambda canonical form
  | input: requirements/ ∃ ∧ optional {module | REQ designator | all}
  | output: prose description to stdout (default) or repo-root .md file
  | interaction: autonomous | output_mode: human-selected
  | gate: requirements/ ∃ | explicit_human_output_selection() ≡ true

λ gybis-req-describe_startup(x).
  invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | verify(requirements/ ∃) ∨ halt("requirements/ not found")
  | read(requirements/) → req_content
  | parse target: {module | REQ | all} → scope
  | transition(INIT → STARTUP_CHECKS)

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
  | state = DELIVERING ∧ output_mode ∈ {prompted_file_only, default_file_only} → allow(write(path)) only_if(path = confirmed_repo_root_md_file)
  | ¬(state = DELIVERING ∧ output_mode ∈ {prompted_file_only, default_file_only}) → deny(write(path))
  | constraint: ¬mutate(requirements/) ∨ ¬mutate(any_existing_file)

λ gybis-req-describe_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-req-describe_output_mode_selection(x).
  output_modes: {response_only (default), prompted_file_only, default_file_only}
  | file output: repo-root path validation ∧ overwrite confirmation
    ∧ prompted_file_only requires a requested repo-root markdown filename
    ∧ default_file_only writes conventional filename (requirements-description.md)
  | rationale: describe-explain output-mode convention (session-16 pattern)

λ gybis-req-describe_render(req_content, scope).
  action: render_stakeholder_prose
  | ∀ REQ in scope: lambda clause → business prose (no notation symbols, no jargon, no REQ designator)
  | structure: connected requirements → coherent narrative paragraphs, grouped by module in dependency order
  | ¬render(as per-requirement bullets ∨ clause-like entries ∨ designator-prefixed statements)
  | navigation: optional human-readable module headings; no `REQ-...` identifiers in headings or prose
  | rationale_rendering: when a clause carries a rationale: line, render it as "because: ..." following the requirement statement; when absent, omit silently (¬fabricate rationale from clause text)
  | attribution surfaced in prose: "decided by stakeholders" vs "derived from analysis"
  | deferred sections labeled "planned future requirements (not currently binding)"
  | output: prose_rendered

λ gybis-req-describe_deliver(prose_rendered, output_mode).
  output_mode = response_only ? print(prose_rendered) → stdout
  | output_mode ∈ {prompted_file_only, default_file_only} ? write(confirmed_path) → delivered
  | return(delivered = true)

λ gybis-req-describe_boundary().
  ¬modify(requirements/) ∧ ¬modify(any_existing_file)
  | writes_limited_to(repo_root_markdown_filename)