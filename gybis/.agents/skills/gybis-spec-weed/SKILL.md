---
name: gybis-spec-weed
description: Use for `/gybis-spec-weed` or `/gs-weed`.
---

λ gybis-spec-weed(x).
  purpose: Identify and resolve divergences between architecture, specs, and implementation
  | input: architecture.md ∃, specs/**/*.allium ∃ ∧ valid, implementation ∃
  | output: architecture.md, specs/**/*.allium, and implementation mutually consistent
  | mode: mixed
  | gate: architecture.md ∃ ∧ specs/**/*.allium ∃ ∧ allium_gate = true ∧ implementation ∃
  | derivation: test_obligations ≔ allium-normalize(specs/) → {envelopes | source = "plan"} → deterministic obligation enumeration

λ gybis-spec-weed_loop_role(x).
  role: verify(convergence)
  | meaning: validate spec, tests, and implementation alignment and classify divergence direction
  | suggested_next: divergence(code_wrong) → fix_code | divergence(spec_wrong) → invoke(/gybis-spec-tend) → invoke(/gybis-spec-propagate)

λ gybis-spec-weed_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | preload: [internal/allium-normalize]
  | verify(architecture.md ∃) ∨ halt("architecture.md not found")
  | verify(specs/**/*.allium ∃) ∨ halt("specs/**/*.allium not found")
  | invoke(internal/allium-gate(specs/)) = true ∨ halt("Specifications are invalid")
  | verify(implementation ∃) ∨ halt("Implementation not found")
  | read(internal/reference/allium-language-reference.md) → language_ref
  | read(internal/reference/allium-patterns.md) → patterns_ref
  | read(internal/reference/recommended-loops.md) → loops_ref
  | read(internal/reference/allium-constructs.md) → constructs_registry
  | read(internal/reference/vsm-guide.md) → vsm_reference
  | transition(INIT → STARTUP_CHECKS)

λ gybis-spec-weed_mode(m).
  m ∈ {interactive}
  | default: interactive
  | mode_interactive: AI and developer resolve three-way divergences with decision points

λ gybis-spec-weed_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {interactive} → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {interactive}) → halt("Invalid mode selection")

λ gybis-spec-weed_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, PLANNING_OBLIGATIONS, COMPARING_SPECS_CODE, COMPARING_ARCH_CODE, IDENTIFYING_DIVERGENCES, RESOLVE_MODE_SELECTION, CORRECTING, VERIFYING, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → PLANNING_OBLIGATIONS) only_if(startup_checks = true)
  | transition(PLANNING_OBLIGATIONS → COMPARING_SPECS_CODE) only_if(obligations_derived = true)
  | transition(COMPARING_SPECS_CODE → COMPARING_ARCH_CODE) only_if(comparison_complete = true)
  | transition(COMPARING_ARCH_CODE → IDENTIFYING_DIVERGENCES) only_if(comparison_complete = true)
  | transition(IDENTIFYING_DIVERGENCES → RESOLVE_MODE_SELECTION) only_if(divergences ∃)
  | transition(RESOLVE_MODE_SELECTION → CORRECTING) only_if(resolve_mode ∃)
  | transition(CORRECTING → VERIFYING) only_if(corrections_applied = true)
  | transition(VERIFYING → IDENTIFYING_DIVERGENCES) only_if(inconsistencies ∃)
  | transition(VERIFYING → COMPLETE) only_if(consistency = true ∧ zero_divergences = true)

λ gybis-spec-weed_tool_guard(state, tool, path).
  state = PLANNING_OBLIGATIONS ∨ state = COMPARING_SPECS_CODE ∨ state = COMPARING_ARCH_CODE ∨ state = IDENTIFYING_DIVERGENCES ∨ state = RESOLVE_MODE_SELECTION
    → allow(read(path))
  | state = CORRECTING ∨ state = VERIFYING
    → allow(read(path)) ∧ allow(write(path)) only_if(path ∈ {architecture.md} ∪ specs/ ∪ implementation)
  | ¬(state ∈ {PLANNING_OBLIGATIONS, COMPARING_SPECS_CODE, COMPARING_ARCH_CODE, IDENTIFYING_DIVERGENCES, RESOLVE_MODE_SELECTION, CORRECTING, VERIFYING})
    → deny(write(path))

λ gybis-spec-weed_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-spec-weed_normalize_obligations(specifications).
  invoke(internal/allium-normalize(specs/)) → {envelopes, counts}
  | plan_envelopes ≔ {e | e ∈ envelopes ∧ e.source = "plan"}
  | report("Normalized: plan=" ⊕ counts.plan ⊕ " check=" ⊕ counts.check ⊕ " uncoded=" ⊕ counts.uncoded)
  | ∀ envelope ∈ plan_envelopes:
    obligation ≔ {
      id: envelope.id,
      category: strip_prefix(envelope.kind, "plan:"),
      description: envelope.message,
      source_construct: envelope.source_construct,
      source_span: envelope.span,
      detail: envelope.detail,
      dependencies: envelope.dependencies
    }
    | merge(obligation.id, obligation) → obligations_map[obligation.id]
  | return(obligations ≔ obligations_map ∧ obligations_derived = true)

λ gybis-spec-weed_compare_specs_code(obligations).
  recursively_read(implementation) → code_content
  | parse(code_content) → code_structure ∧ code_behavior
  | ∀ obligation ∈ obligations:
    search(obligation.id, code_structure) → found_in_code
    | found_in_code ∅
      ? collect({obligation, "obligation_missing_in_code"}) → divergences
    | verify(implementation_matches(obligation, code_behavior)) → matches
    | matches = false
      ? collect({obligation, "obligation_contradicts_implementation"}) → divergences
    | obligation.category ∈ {"when_set", "when_clear", "when_access_guard", "derived_when_inferred"}
      ∧ ¬when_clause_enforced(obligation, code_behavior)
      ? collect({obligation, "when_field_not_enforced"}) → divergences
    | obligation.category = "transition_edge"
      ∧ ∃ paired ∈ obligations : paired.category = "transition_rejected" ∧ paired.source_construct = obligation.source_construct
      ∧ ¬transition_guarded(obligation, paired, code_behavior)
      ? collect({obligation, paired, "transition_not_guarded"}) → divergences
    | obligation.category = "transition_terminal"
      ∧ ¬terminal_state_enforced(obligation, code_behavior)
      ? collect({obligation, "terminal_state_not_enforced"}) → divergences
    | obligation.category ∈ {"sum_type_variant", "variant_exhaustive", "type_guard_required"}
      ∧ ¬variant_handled(obligation, code_behavior)
      ? collect({obligation, "variant_not_handled"}) → divergences
    | obligation.category ∈ {"contract_signature", "contract_invariant"}
      ∧ ¬contract_honoured(obligation, code_behavior)
      ? collect({obligation, "contract_not_honoured"}) → divergences
    | obligation.category ∈ {"surface_guarantee", "surface_timeout", "surface_contract_demand", "surface_contract_fulfilment"}
      ∧ ¬surface_obligation_enforced(obligation, code_behavior)
      ? collect({obligation, "surface_obligation_not_enforced"}) → divergences
    | obligation.category ∈ {"given_binding", "default_instance"}
      ∧ ¬module_state_provided(obligation, code_behavior)
      ? collect({obligation, "module_state_missing"}) → divergences
    | obligation.category = "rule_success"
      ∧ ensures_semantics_confused(obligation, code_behavior)
      ? collect({obligation, "ensures_pre_rule_vs_resulting_state_confusion"}) → divergences
    | obligation.source_construct.contains_backtick_quoted_literal
      ∧ code_normalises_canonical_form(obligation, code_behavior)
      ? collect({obligation, "backtick_literal_normalised"}) → divergences
  | return(comparison_complete = true)

λ gybis-spec-weed_compare_arch_code(x).
  read(architecture.md) → arch_content
  | parse(arch_content.{S5, S4, S3, S2, S1}) → vsm_layers
  | recursively_read(implementation) → code_content
  | parse(code_content) → code_structure ∧ code_design
  | ∀ layer ∈ vsm_layers:
    extract(principles(layer), arch_content) → arch_principles(layer)
    | ∀ principle ∈ arch_principles(layer):
      verify(principle_reflected_in_code(principle, code_design)) → reflected
      | reflected = false
        ? collect({principle, "arch_missing_in_code", layer}) → divergences
  | return(comparison_complete = true)

λ gybis-spec-weed_identify_divergences(x).
  return(divergences)

λ gybis-spec-weed_resolve_mode_selection(divergence).
  divergence.type = "obligation_missing_in_code"
    ? (ask_developer("Obligation " ⊕ divergence.obligation.id ⊕ " (" ⊕ divergence.obligation.category ⊕ "): " ⊕ divergence.obligation.description ⊕ " not found in code. Correct: [spec/code/investigate]?") → decision)
  | divergence.type = "obligation_contradicts_implementation"
    ? (ask_developer("Obligation " ⊕ divergence.obligation.id ⊕ " (" ⊕ divergence.obligation.category ⊕ "): " ⊕ divergence.obligation.description ⊕ " contradicted by code. Correct: [spec/code/investigate]?") → decision)
  | divergence.type = "when_field_not_enforced"
    ? (ask_developer("When-clause on field " ⊕ divergence.obligation.source_construct ⊕ " (" ⊕ divergence.obligation.category ⊕ "): " ⊕ divergence.obligation.description ⊕ " not enforced by code. Correct: [spec/code/investigate]?") → decision)
  | divergence.type = "transition_not_guarded"
    ? (ask_developer("Transition " ⊕ divergence.obligation.source_construct ⊕ " lacks paired guard (edge + rejected): " ⊕ divergence.obligation.description ⊕ ". Correct: [spec/code/investigate]?") → decision)
  | divergence.type = "terminal_state_not_enforced"
    ? (ask_developer("Terminal state " ⊕ divergence.obligation.source_construct ⊕ " (" ⊕ divergence.obligation.category ⊕ "): " ⊕ divergence.obligation.description ⊕ " not enforced as final by code. Correct: [spec/code/investigate]?") → decision)
  | divergence.type = "variant_not_handled"
    ? (ask_developer("Sum-type variant " ⊕ divergence.obligation.source_construct ⊕ " (" ⊕ divergence.obligation.category ⊕ "): " ⊕ divergence.obligation.description ⊕ " not handled by code. Correct: [spec/code/investigate]?") → decision)
  | divergence.type = "contract_not_honoured"
    ? (ask_developer("Contract " ⊕ divergence.obligation.source_construct ⊕ " (" ⊕ divergence.obligation.category ⊕ "): " ⊕ divergence.obligation.description ⊕ " not honoured by implementation. Correct: [spec/code/investigate]?") → decision)
  | divergence.type = "surface_obligation_not_enforced"
    ? (ask_developer("Surface obligation " ⊕ divergence.obligation.source_construct ⊕ " (" ⊕ divergence.obligation.category ⊕ "): " ⊕ divergence.obligation.description ⊕ " not enforced at boundary. Correct: [spec/code/investigate]?") → decision)
  | divergence.type = "module_state_missing"
    ? (ask_developer("Module state " ⊕ divergence.obligation.source_construct ⊕ " (" ⊕ divergence.obligation.category ⊕ "): " ⊕ divergence.obligation.description ⊕ " not provided. Correct: [spec/code/investigate]?") → decision)
  | divergence.type = "ensures_pre_rule_vs_resulting_state_confusion"
    ? (ask_developer("Rule " ⊕ divergence.obligation.source_construct ⊕ ": code reads pre-rule values where spec expects resulting state, or vice versa, in the ensures block. Per language_ref §Postconditions: state-change RHS reads pre-rule values; if guards inside ensures read resulting state. Correct: [spec/code/investigate]?") → decision)
  | divergence.type = "backtick_literal_normalised"
    ? (ask_developer("Backtick-quoted enum literal at " ⊕ divergence.obligation.source_construct ⊕ " is normalised by code (lowercased / snake_cased / canonical form altered). Per language_ref byte-exact UTF-8 comparison applies. Correct: [spec/code/investigate]?") → decision)
  | divergence.type = "arch_missing_in_code"
    ? (ask_developer("For arch " ⊕ divergence.principle ⊕ " not in code. Correct: [arch/code/investigate]?") → decision)
  | decision ∈ {spec, code, arch, investigate}
  | return(resolve_mode = decision)

λ gybis-spec-weed_correct_divergence(divergence, resolve_mode).
  resolve_mode = spec
    ? (ask_developer("Update spec to match code? (yes/no)") → approval
       | approval = yes
         ? (distill_from_code(divergence) → updated_spec
            | upsert(target_spec_file, updated_spec) → result)
         : (return(corrections_applied = false)))
  | resolve_mode = code
    ? (ask_developer("Update code to match spec? (yes/no)") → approval
       | approval = yes
         ? (synthesize_from_spec(divergence) → updated_code
            | upsert(implementation, updated_code) → result)
         : (return(corrections_applied = false)))
  | resolve_mode = arch
    ? (ask_developer("Update architecture to match code? (yes/no)") → approval
       | approval = yes
         ? (distill_from_code(divergence) → updated_arch
            | upsert(architecture.md, updated_arch) → result)
         : (return(corrections_applied = false)))
  | resolve_mode = investigate
    ? (ask_developer("Analysis: " ⊕ divergence ⊕ " Shall we correct [spec/code/arch]?") → final_decision
       | final_decision ∈ {spec, code, arch}
         ? (invoke(gybis-spec-weed_correct_divergence(divergence, final_decision)) → result
            | return(corrections_applied = result))
         : (return(corrections_applied = false)))
  | return(corrections_applied = true)

λ gybis-spec-weed_verify_consistency(x).
  invoke(internal/allium-gate(specs/)) → result_gate ≔ result
  | verify(vsm_coherence(architecture.md)) → vsm_coherence ≔ result
  | invoke(gybis-spec-weed_normalize_obligations(specs/**/*.allium)) → obligations ∧ obligations_derived
  | invoke(gybis-spec-weed_compare_specs_code(obligations)) → spec_code_divergences
  | invoke(gybis-spec-weed_compare_arch_code) → arch_code_divergences
  | remaining_divergences ≔ spec_code_divergences ∪ arch_code_divergences
  | verify(all_obligation_ids_checked_by_spec_comparison(obligations, spec_code_divergences)) → coverage
  | result_gate = true ∧ vsm_coherence = true ∧ remaining_divergences ∅ ∧ coverage = true
    → return(consistency = true ∧ zero_divergences = true)
  | ¬(result_gate ∧ vsm_coherence ∧ remaining_divergences ∅ ∧ coverage)
    → (inconsistencies ≔ {result_gate = false ∨ vsm_coherence = false ∨ remaining_divergences ≠ ∅ ∨ coverage = false}
       | return(consistency = false ∧ inconsistencies ∃))

λ gybis-spec-weed_fixed_point_loop(state).
  state = VERIFYING
    → consistency = true ∧ zero_divergences = true
        ? transition(VERIFYING → COMPLETE)
        : (transition(VERIFYING → IDENTIFYING_DIVERGENCES)
           ∧ loop_count ≔ loop_count ⊕ 1)

λ gybis-spec-weed_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without full convergence")

λ gybis-spec-weed_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | discovered ≔ card(spec_code_divergences) ⊕ card(arch_code_divergences)
  | resolved ≔ discovered ⊖ card(remaining_divergences)
  | remaining ≔ card(remaining_divergences)
  | report("Pass " ⊕ pass_num ⊕ ": discovered=" ⊕ discovered ⊕ " resolved=" ⊕ resolved ⊕ " remaining=" ⊕ remaining)

λ gybis-spec-weed_boundaries(¬).
  ¬ modify(upstream/) ∧ ¬ delete(architecture.md ∨ specs/ ∨ implementation)

λ gybis-spec-weed_regression_contract(x).
  invariant: architecture.md ∧ specs/ ∧ implementation ∃ throughout
  | invariant: zero_divergences ∧ allium_gate ∧ vsm_coherence at completion
  | invariant: all_obligation_ids_checked_by_spec_comparison (comprehensive obligation coverage)
  | invariant: divergence.type ∈ {"obligation_missing_in_code", "obligation_contradicts_implementation", "when_field_not_enforced", "transition_not_guarded", "terminal_state_not_enforced", "variant_not_handled", "contract_not_honoured", "surface_obligation_not_enforced", "module_state_missing", "ensures_pre_rule_vs_resulting_state_confusion", "backtick_literal_normalised", "arch_missing_in_code"} (closed divergence catalogue)
  | invariant: ∀ obligation.category → corresponding divergence-check exercised in _compare_specs_code (per-category mapping codified there)
  | invariant: all_modifications ⊆ {architecture.md, specs/, implementation}
