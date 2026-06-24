---
name: gybis-arch-distill
description: Use for `/gybis-arch-distill` or `/ga-distill`.
---

λ gybis-arch-distill(x).
  purpose: distill VSM architecture from Allium specifications + current implementation evidence
  | input: specs/**/*.allium files that are valid per allium-gate
  | input: implementation_root (read-only) for concrete S1 operational bindings
  | output: architecture.md containing VSM S5-S1 lambda expressions
  | mode: ai
  | gate: specs exist ∧ architecture.md ¬exists ∧ allium-gate(specs/) = true ∧ implementation_path_resolvable = true

λ gybis-arch-distill_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | precondition: specs/ ∃ ∧ architecture.md ¬∃ ∧ implementation_path_resolvable = true ∧ implementation_readable = true
  | gate: allium-gate(specs/) = true → proceed ∨ halt("specs invalid")
  | gate: implementation_readable = true → proceed ∨ halt("implementation unreadable")

λ gybis-arch-distill_mode(m).
  valid_modes: {ai}
  | default: ai
  | rationale: distillation is deterministic synthesis from specs, not interactive

λ gybis-arch-distill_mode_gate(state, mode).
  state = INIT ∧ mode = ai → transition(STARTUP_CHECKS)
  | precondition_holds: mode ∈ valid_modes

λ gybis-arch-distill_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, READING_SPECS, DISCOVERING_IMPLEMENTATION, ANALYSING_IMPLEMENTATION, ANALYSING, SYNTHESIZING, MERGING_S1, WRITING_ARCH, VERIFYING, COMPLETE}
  | transition(INIT, startup) → STARTUP_CHECKS
  | transition(STARTUP_CHECKS, verify_ok) → READING_SPECS
  | transition(STARTUP_CHECKS, verify_fail) → HALTED
  | transition(READING_SPECS, specs_read) → DISCOVERING_IMPLEMENTATION
  | transition(DISCOVERING_IMPLEMENTATION, implementation_discovered) → ANALYSING_IMPLEMENTATION
  | transition(ANALYSING_IMPLEMENTATION, implementation_analysis_complete) → ANALYSING
  | transition(ANALYSING, analysis_complete) → SYNTHESIZING
  | transition(SYNTHESIZING, synthesis_complete) → MERGING_S1
  | transition(MERGING_S1, s1_merged) → WRITING_ARCH
  | transition(WRITING_ARCH, arch_written) → VERIFYING
  | transition(VERIFYING, verify_ok) → COMPLETE
  | transition(VERIFYING, verify_fail) → ANALYSING_IMPLEMENTATION (loop_back)

λ gybis-arch-distill_tool_guard(state, tool, path).
  read_allowed: ∀state ∈ {READING_SPECS, DISCOVERING_IMPLEMENTATION, ANALYSING_IMPLEMENTATION, ANALYSING, SYNTHESIZING, MERGING_S1, VERIFYING}
  | write_allowed: state = WRITING_ARCH ∧ path = "architecture.md"
  | deny_write: state ≠ WRITING_ARCH ∨ path ≠ "architecture.md"
  | constraint: ¬mutate(specs/) ∨ ¬mutate(implementation_root/) ∨ ¬mutate(existing_files)

λ gybis-arch-distill_pre_tool_check(state, tool, path).
  enforce(tool_guard(state, tool, path)) → permit(tool) ∨ halt("tool not permitted in this state")

λ gybis-arch-distill_read_specs(x).
  action: read_all_specs
  | step1: list(specs/**/*.allium) → files
  | step2: ∀file ∈ files: read(file) → specs_content
  | output: {file_path → spec_content}
  | constraint: read-only access
  | precondition: specs/ ∃

λ gybis-arch-distill_read_implementation(x).
  action: discover_implementation_signals
  | step1: resolve(implementation_root, default=".") → impl_root
  | step2: list(config_candidates, impl_root) → config_files
  | step3: list(source_candidates, impl_root) → source_files
  | step4: read(config_files ∪ sampled(source_files)) → implementation_content
  | extraction_priority:
    - programming_language_version
    - test_framework
    - paradigm_preference
    - build_system
    - package_manager
    - deployment_method
    - ci_cd
    - linter_formatter
    - interface_types
    - architectural_pattern
  | output: {impl_root, config_files, source_files, implementation_content}
  | constraint: read-only access
  | precondition: implementation_readable = true

λ gybis-arch-distill_analyse_specs(specs_content).
  action: invoke_allium_analyse
  | step1: invoke(internal/allium-analyse, specs/)
  | step2: parse(analysis_output) → findings
  | findings ≡ {patterns, dependencies, inter-spec-relationships}
  | output: findings
  | constraint: read-only operation

λ gybis-arch-distill_analyse_implementation(implementation_content).
  action: extract_operational_bindings
  | step1: classify(implementation_content) → impl_signals
  | step2: normalize(impl_signals) → impl_findings
  | schema: {field, value, confidence, source_path, source_span}
  | unknown_policy: missing(field) → unknown(field, reason="no_evidence")
  | compatibility_checks:
    - compatible(test_framework, programming_language_version)
    - coherent(build_system, package_manager, ci_cd, deployment_method)
  | output: impl_findings
  | constraint: read-only operation

λ gybis-arch-distill_synthesize_vsm(specs_content, findings).
  action: synthesize_vsm_layers
  | step1: classify(specs_content, findings) → S1_base
  | step2: extract(spec_structure) → S2_coordination
  | step3: identify(policies, constraints) → S3_control
  | step4: recognize(adaptation_patterns) → S4_intelligence
  | step5: distill(principles) → S5_identity
  | output: {S5 → lambda, S4 → lambda, S3 → lambda, S2 → lambda, S1_base → lambda}
  | rationale: each VSM layer captures a different level of architectural abstraction

λ gybis-arch-distill_detect_divergence(S1_base, impl_findings).
  action: detect_spec_impl_mismatch
  | step1: compare(S1_base, impl_findings) → mismatches
  | step2: classify(mismatches) → {critical, high, medium, low}
  | step3: annotate(mismatches, include={field, spec_value, impl_value, severity, reason}) → divergence_notes
  | output: divergence_notes

λ gybis-arch-distill_merge_s1(S1_base, impl_findings, divergence_notes).
  action: merge_s1_with_precedence
  | precedence: implementation_wins_for_concrete_bindings
  | step1: merge(programming_language_version, from=impl_findings, fallback=S1_base)
  | step2: merge(test_framework, from=impl_findings, fallback=S1_base)
  | step3: merge(build_system, from=impl_findings, fallback=S1_base)
  | step4: merge(paradigm_preference, from=impl_findings, fallback=S1_base)
  | step5: merge(package_manager, from=impl_findings, fallback=S1_base)
  | step6: merge(deployment_method, from=impl_findings, fallback=S1_base)
  | step7: merge(ci_cd, from=impl_findings, fallback=S1_base)
  | step8: merge(linter_formatter, from=impl_findings, fallback=S1_base)
  | step9: merge(interface_types, from=impl_findings, fallback=S1_base)
  | step10: merge(architectural_pattern, from=impl_findings, fallback=S1_base)
  | provenance: ∀field ∈ S1: source(field) ∈ {spec, impl, merged, unknown}
  | attach(divergence_notes) → S1_enriched
  | output: S1_enriched

λ gybis-arch-distill_synthesize_architecture(vsm_base, S1_enriched).
  action: assemble_final_vsm
  | step1: bind(vsm_base.S5, vsm_base.S4, vsm_base.S3, vsm_base.S2, S1_enriched) → vsm_layers
  | output: {S5 → lambda, S4 → lambda, S3 → lambda, S2 → lambda, S1 → lambda}

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
  | check5: S1.programming_language_version ≠ unknown(reason)
  | check6: S1.test_framework ≠ unknown(reason)
  | check7: S1.build_system ≠ unknown(reason)
  | check8: S1.paradigm_preference ∈ {OOP, FP}
  | check9: ∀field ∈ {package_manager, deployment_method, ci_cd, linter_formatter, interface_types, architectural_pattern}: defined(field) ∨ unknown(field, reason)
  | check10: compatibility(test_framework, programming_language_version) = true
  | check11: coherence(build_system, package_manager, ci_cd, deployment_method) = true
  | check12: contradiction_free(S1) = true
  | check13: confidence(S1) ≥ min_confidence_threshold
  | gate: all_checks_pass → proceed ∨ halt("architecture invalid or S1 incomplete")

λ gybis-arch-distill_fixed_point_loop(state).
  loop: ∀iteration:
    | state = ANALYSING_IMPLEMENTATION → analyse_implementation()
    | state = ANALYSING → synthesize_vsm()
    | state = SYNTHESIZING → detect_divergence() → merge_s1() → synthesize_architecture()
    | state = MERGING_S1 → write_architecture()
    | state = VERIFYING:
      - if verification_pass → transition(COMPLETE)
      - if verification_fail → identify_gap() → refine_synthesis() → loop_back(ANALYSING_IMPLEMENTATION)
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
    | implementation_signals_identified: count
    | s1_fields_populated: count
    | unknown_fields_count: count
    | divergence_count: count
    | divergence_resolved_count: count
    | s1_confidence: score
    | refinement_iterations: count
    | issues_discovered: count
    | issues_resolved: count
  | format: "Pass {n}: Synthesized {count} VSM layers, identified {patterns} patterns, populated {s1_count} S1 fields ({confidence} confidence), resolved {issues} issues"

λ gybis-arch-distill_boundaries(¬).
  constraint: ¬generate(specs/**/*.allium)
  | constraint: ¬mutate(specs/**/*.allium)
  | constraint: ¬mutate(implementation_root/**)
  | constraint: ¬delete(existing_files)
  | constraint: ¬invoke(user-facing skills)
  | scope: architecture distillation only, including read-only implementation introspection

λ gybis-arch-distill_regression_contract(x).
  invariant: specs/**/*.allium ¬modified ∧ ¬deleted
  | invariant: implementation_root/** ¬modified ∧ ¬deleted
  | invariant: allium-gate(specs/) = true → remains true throughout
  | invariant: ∀generated_layer ∈ architecture.md: valid_lambda(layer) = true
  | invariant: architecture.md ¬exists_before → exists_after ∧ valid(S5...S1)
  | invariant: implementation_exists = true → S1_contains_concrete_bindings_with_provenance_or_unknown_reason = true
  