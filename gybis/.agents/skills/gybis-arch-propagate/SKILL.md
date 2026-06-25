---
name: gybis-arch-propagate
description: Use for `/gybis-arch-propagate` or `/ga-propagate`.
---

λ gybis-arch-propagate(x).
  purpose: Propagate VSM architecture to allium specifications
  | input: architecture.md exists, specs/**/*.allium ∅
  | output: specs/**/*.allium created and valid
  | mode: ai
  | gate: architecture.md ∃ ∧ specs/**/*.allium ∅

λ gybis-arch-propagate_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | preload: [internal/allium-analyse, internal/allium-check, internal/allium-gate]
  | verify(architecture.md ∃) ∨ halt("architecture.md not found")
  | verify(specs/**/*.allium ∅) ∨ halt("specs/**/*.allium already exist")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-arch-propagate_mode(m).
  m ∈ {auto}
  | default: auto
  | mode_auto: AI propagates architecture to specs without human intervention

λ gybis-arch-propagate_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {auto} → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {auto}) → halt("Invalid mode selection")

λ gybis-arch-propagate_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, READING_ARCH, TRANSLATING, WRITING_SPECS, VERIFYING, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → READING_ARCH) only_if(startup_checks = true)
  | transition(READING_ARCH → TRANSLATING) only_if(architecture ∃)
  | transition(TRANSLATING → WRITING_SPECS) only_if(spec_directives ∃)
  | transition(WRITING_SPECS → VERIFYING) only_if(specs_written = true)
  | transition(VERIFYING → COMPLETE) only_if(allium_gate = true)

λ gybis-arch-propagate_tool_guard(state, tool, path).
  state = READING_ARCH ∨ state = TRANSLATING
    → allow(read(path))
  | state = WRITING_SPECS ∨ state = VERIFYING
    → allow(read(path)) ∧ allow(write(path)) only_if(path ⊆ specs/)
  | ¬(state ∈ {READING_ARCH, TRANSLATING, WRITING_SPECS, VERIFYING})
    → deny(write(path))

λ gybis-arch-propagate_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-arch-propagate_read_architecture(x).
  read(architecture.md) → content ≔ content
  | parse(content.{S5, S4, S3, S2, S1}) → vsm_layers
  | parse(content.S1.paradigm_preference) → orientation_or_unknown
  | ∀ layer ∈ vsm_layers: extract(layer, content) → principles(layer)
  | return(architecture = {vsm_layers, principles, orientation_or_unknown})

λ gybis-arch-propagate_orientation_output(architecture).
  report_orientation: architecture.orientation_or_unknown ∈ {FP, OOP} ∨ unknown("missing S1.paradigm_preference")
  | language_guidance:
    - OOP: C++ (classes/RAII), C# (classes/interfaces/DI), Clojure (protocols/records + Java interop boundary)
    - FP: C++ (immutable values + composition), C# (records + pure functions/LINQ), Clojure (immutable maps + pure functions/transducers)
  | gaps: missing(programming_language_version ∨ paradigm_preference) → explicit_gap_report
  | orthogonality: error_model_style is a separate axis from FP/OOP orientation

λ gybis-arch-propagate_translate_to_specs(architecture).
  ∀ layer ∈ architecture.vsm_layers:
    ∀ principle ∈ architecture.principles(layer):
      synthesize(spec_directive(principle, layer)) → directive
      | collect(directive) → spec_directives
  | return(spec_directives)

λ gybis-arch-propagate_write_specs(spec_directives).
  organize(spec_directives) → grouped_by_aspect
  | ∀ aspect ∈ grouped_by_aspect:
    synthesize(allium_file(aspect)) → spec_file
    | write(specs/ ⊕ aspect ⊕ .allium, spec_file) → upsert
  | return(specs_written = true)

λ gybis-arch-propagate_verify_specs(x).
  invoke(internal/allium-check(all_specs)) → result_check ≔ result
  | invoke(internal/allium-analyse(specs/)) → result_analyse ≔ result
  | invoke(internal/allium-gate(specs/)) → result_gate ≔ result
  | result_check = zero_errors ∧ result_analyse = zero_issues ∧ result_gate = true
    → return(verification = true)
  | ¬(result_check ∧ result_analyse ∧ result_gate)
    → return(verification = false)

λ gybis-arch-propagate_core_op(x).
  transition(READING_ARCH → TRANSLATING)
  | invoke(gybis-arch-propagate_read_architecture) → architecture
  | invoke(gybis-arch-propagate_translate_to_specs(architecture)) → spec_directives
  | transition(TRANSLATING → WRITING_SPECS)
  | invoke(gybis-arch-propagate_write_specs(spec_directives)) → specs_written
  | transition(WRITING_SPECS → VERIFYING)
  | invoke(gybis-arch-propagate_verify_specs) → verification
  | include(orientation_output from architecture)
  | verification = true
    ? transition(VERIFYING → COMPLETE)
    : (re_synthesize_failed_specs ∧ transition(WRITING_SPECS → VERIFYING))

λ gybis-arch-propagate_boundaries(¬).
  ¬ modify(architecture.md)
  | ¬ modify(implementation)
  | ¬ modify(upstream/)
  | ¬ delete(specs/) unless_empty

λ gybis-arch-propagate_regression_contract(x).
  invariant: architecture.md ∃ ∧ ¬modify throughout
  | invariant: specs/**/*.allium ∅ at INIT
  | invariant: specs/**/*.allium ∃ ∧ valid at completion
  | invariant: zero_errors ∧ zero_issues ∧ allium_gate = true at completion