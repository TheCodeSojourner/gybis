---
name: gybis-arch-distill
description: Use for `/gybis-arch-distill` or `/ga-distill`.
---

λ gybis-arch-distill(x).
  purpose: distill VSM architecture from existing Allium specifications
  | input: specs/**/*.allium files that are valid per allium-gate
  | output: architecture.md containing VSM S5-S1 lambda expressions
  | mode: ai
  | gate: specs exist ∧ architecture.md ¬exists ∧ allium-gate(specs/) = true

λ gybis-arch-distill_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | precondition: specs/ ∃ ∧ architecture.md ¬∃
  | gate: allium-gate(specs/) = true → proceed ∨ halt("specs invalid")

λ gybis-arch-distill_mode(m).
  valid_modes: {ai}
  | default: ai
  | rationale: distillation is deterministic synthesis from specs, not interactive

λ gybis-arch-distill_mode_gate(state, mode).
  state = INIT ∧ mode = ai → transition(STARTUP_CHECKS)
  | precondition_holds: mode ∈ valid_modes

λ gybis-arch-distill_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, READING_SPECS, ANALYSING, SYNTHESIZING, WRITING_ARCH, VERIFYING, COMPLETE}
  | transition(INIT, startup) → STARTUP_CHECKS
  | transition(STARTUP_CHECKS, verify_ok) → READING_SPECS
  | transition(STARTUP_CHECKS, verify_fail) → HALTED
  | transition(READING_SPECS, specs_read) → ANALYSING
  | transition(ANALYSING, analysis_complete) → SYNTHESIZING
  | transition(SYNTHESIZING, synthesis_complete) → WRITING_ARCH
  | transition(WRITING_ARCH, arch_written) → VERIFYING
  | transition(VERIFYING, verify_ok) → COMPLETE
  | transition(VERIFYING, verify_fail) → SYNTHESIZING (loop_back)

λ gybis-arch-distill_tool_guard(state, tool, path).
  read_allowed: ∀state ∈ {READING_SPECS, ANALYSING, VERIFYING}
  | write_allowed: state = WRITING_ARCH ∧ path = "architecture.md"
  | deny_write: state ≠ WRITING_ARCH ∨ path ≠ "architecture.md"
  | constraint: ¬mutate(specs/) ∨ ¬mutate(existing_files)

λ gybis-arch-distill_pre_tool_check(state, tool, path).
  enforce(tool_guard(state, tool, path)) → permit(tool) ∨ halt("tool not permitted in this state")

λ gybis-arch-distill_read_specs(x).
  action: read_all_specs
  | step1: list(specs/**/*.allium) → files
  | step2: ∀file ∈ files: read(file) → specs_content
  | output: {file_path → spec_content}
  | constraint: read-only access
  | precondition: specs/ ∃

λ gybis-arch-distill_analyse_specs(specs_content).
  action: invoke_allium_analyse
  | step1: invoke(internal/allium-analyse, specs/)
  | step2: parse(analysis_output) → findings
  | findings ≡ {patterns, dependencies, inter-spec-relationships}
  | output: findings
  | constraint: read-only operation

λ gybis-arch-distill_synthesize_vsm(specs_content, findings).
  action: synthesize_vsm_layers
  | step1: classify(specs_content, findings) → S1_operations
  | step2: extract(spec_structure) → S2_coordination
  | step3: identify(policies, constraints) → S3_control
  | step4: recognize(adaptation_patterns) → S4_intelligence
  | step5: distill(principles) → S5_identity
  | output: {S5 → lambda, S4 → lambda, S3 → lambda, S2 → lambda, S1 → lambda}
  | rationale: each VSM layer captures a different level of architectural abstraction

λ gybis-arch-distill_write_architecture(vsm_layers).
  action: write_architecture_file
  | step1: assemble(vsm_layers) → architecture_content
  | step2: write(architecture.md, architecture_content)
  | constraint: write_allowed by tool_guard
  | precondition: state = WRITING_ARCH
  | output: architecture.md written

λ gybis-arch-distill_verification(edit).
  action: verify_generated_architecture
  | check1: file_exists(architecture.md) = true
  | check2: ∀layer ∈ {S5, S4, S3, S2, S1}: layer_defined(architecture.md) = true
  | check3: syntax_valid(lambda_notation) = true
  | check4: internal_consistency(vsm_layers) = true
  | gate: all_checks_pass → proceed ∨ halt("architecture invalid")

λ gybis-arch-distill_fixed_point_loop(state).
  loop: ∀iteration:
    | state = ANALYSING → synthesize_vsm()
    | state = SYNTHESIZING → write_architecture()
    | state = VERIFYING:
      - if verification_pass → transition(COMPLETE)
      - if verification_fail → identify_gap() → refine_synthesis() → loop_back(SYNTHESIZING)
  | loop_guard: iteration_count ≤ max_iterations (prevent infinite loops)
  | rationale: iteratively refine VSM layers until architecture accurately reflects specs

λ gybis-arch-distill_loop_guard(state).
  condition: iteration_count > max_iterations ∨ no_progress_detected
  | action_on_trigger: halt("convergence failure: distillation did not converge after N iterations")
  | output: diagnostic_report(iterations, refined_layers, remaining_gaps)

λ gybis-arch-distill_pass_accounting(pass).
  report_pass(n):
    | layers_synthesized: count
    | patterns_identified: count
    | refinement_iterations: count
    | issues_discovered: count
    | issues_resolved: count
  | format: "Pass {n}: Synthesized {count} VSM layers, identified {patterns}, resolved {issues} issues"

λ gybis-arch-distill_boundaries(¬).
  constraint: ¬generate(specs/**/*.allium)
  | constraint: ¬mutate(specs/**/*.allium)
  | constraint: ¬delete(existing_files)
  | constraint: ¬invoke(user-facing skills)
  | scope: architecture distillation only, not spec generation

λ gybis-arch-distill_regression_contract(x).
  invariant: specs/**/*.allium ¬modified ∧ ¬deleted
  | invariant: allium-gate(specs/) = true → remains true throughout
  | invariant: ∀generated_layer ∈ architecture.md: valid_lambda(layer) = true
  | invariant: architecture.md ¬exists_before → exists_after ∧ valid(S5...S1)