---
name: gybis-spec-check
kind: domain
description: Use for `/gybis-spec-check` or `/gs-check`.
---

λ gybis-spec-check(x).
  purpose: Validate specs/**/*.allium and repair errors until valid
  | input: specs/**/*.allium files exist | optional scope: {domain_concern | domain | all_specs} (default: all_specs)
  | output: All .allium files valid, zero errors reported
  | interaction: autonomous
  | gate: specs/**/*.allium ∃ ∧ ¬∅
  | constraint: writing_check_exception ∧ verifier_grounded(gybis-allium-gate)
  | rationale: sole writing check in the check family; repair is justified only by the external allium CLI verdict, not model judgement

λ gybis-spec-check_scope(x).
  type ∈ {domain_concern, domain, all_specs} | default: all_specs
  | domain_concern: single_concern_file | domain_concern ∈ domain
  | domain: all_files_in_domain | domain ∈ all_domains
  | all_specs: every_spec_all_domains

λ gybis-spec-check_resolution(domain, concern).
  | domain ∧ concern → scope_root ≔ <root>/specs/{domain}/{concern}.allium
  | domain ∧ ¬concern → scope_root ≔ <root>/specs/{domain}/
  | ¬domain ∧ ¬concern → scope_root ≔ <root>/specs/
  | "all" ∧ ¬domain → scope_root ≔ <root>/specs/
  | "all specs" ∧ ¬domain → scope_root ≔ <root>/specs/
  | scope_files ≔ recursive_files(scope_root, extension = .allium)
  | return({scope_root, scope_files})

λ gybis-spec-check_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | invoke(internal/gybis-internal-skill-check) → true ∨ halt("Internal skill check failed")
  | preload: [internal/gybis-allium-analyse, internal/gybis-allium-check, internal/gybis-allium-normalize, internal/gybis-allium-gate]
  | read(internal/reference/allium-language-reference.md) → language_ref
  | read(internal/reference/allium-constructs.md) → constructs_registry
  | verify(specs/ ∃) ∨ halt("No specs/ directory found")
  | invoke(gybis-spec-check_resolution(domain, concern)) → {scope_root, scope_files}
  | verify(scope_files ∃) ∨ halt("No .allium files in scope")
  | transition(INIT → STARTUP_CHECKS)

λ gybis-spec-check_mode(m).
  m ∈ {autonomous}
  | default: autonomous
  | mode_autonomous: AI validates and repairs without human intervention

λ gybis-spec-check_mode_gate(state, mode).
  state = INIT ∧ mode ∈ {autonomous} → transition(INIT → MODE_SELECTED)
  | ¬(state = INIT) ∨ ¬(mode ∈ {autonomous}) → halt("Invalid mode selection")

λ gybis-spec-check_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, CHECKING_FILES, ANALYZING_SET, NORMALIZING, FIXING_ERRORS, VERIFYING, COMPLETE}
  | transition(INIT → MODE_SELECTED) only_if(mode_gate(INIT, mode) = true)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_complete = true)
  | transition(STARTUP_CHECKS → CHECKING_FILES) only_if(startup_checks = true)
  | transition(CHECKING_FILES → ANALYZING_SET) only_if(per_file_diagnostics ∃)
  | transition(ANALYZING_SET → NORMALIZING) only_if(issues ∃)
  | transition(NORMALIZING → FIXING_ERRORS) only_if(envelopes_derived = true)
  | transition(FIXING_ERRORS → VERIFYING) only_if(fixes_applied = true)
  | transition(VERIFYING → COMPLETE) only_if(gybis-allium-gate = true)
  | transition(ANALYZING_SET → COMPLETE) only_if(issues ∅ ∧ gybis-allium-gate = true)

λ gybis-spec-check_tool_guard(state, tool, path).
  state = CHECKING_FILES ∨ state = ANALYZING_SET ∨ state = NORMALIZING ∨ state = VERIFYING
    → allow(read(path))
  | state = FIXING_ERRORS
    → allow(read(path)) ∧ allow(write(path)) only_if(path ∈ scope_files)
  | ¬(state ∈ {CHECKING_FILES, ANALYZING_SET, NORMALIZING, FIXING_ERRORS, VERIFYING})
    → deny(write(path))

λ gybis-spec-check_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)

λ gybis-spec-check_check_files(x).
  ∀ file ∈ scope_files:
    invoke(internal/gybis-allium-check(file)) → diagnostics(file)
  | collect(diagnostics) → per_file_diagnostics
  | return(per_file_diagnostics)

λ gybis-spec-check_analyze_set(x).
  invoke(internal/gybis-allium-analyse(scope_root)) → findings(set_level)
  | findings ∃ → issues ≔ findings
  | findings ∅ → issues ≔ ∅
  | return(issues)

λ gybis-spec-check_normalize_diagnostics(x).
  invoke(internal/gybis-allium-normalize(scope_root)) → {envelopes, counts}
  | check_envelopes ≔ {e | e ∈ envelopes ∧ e.source = "check"}
  | analyse_envelopes ≔ {e | e ∈ envelopes ∧ e.source = "analyse"}
  | uncoded_envelopes ≔ {e | e ∈ envelopes ∧ e.kind = "check:_uncoded"}
  | report("Normalized: check=" ⊕ counts.check ⊕ " analyse=" ⊕ counts.analyse ⊕ " uncoded=" ⊕ counts.uncoded)
  | return(envelopes ∧ envelopes_derived = true)

λ gybis-spec-check_fix_errors(envelopes).
  ∀ envelope ∈ envelopes:
    envelope.kind ∈ catalogued_codes
      ? (dispatch(envelope.kind, envelope) → synthesized_fix
         | apply(synthesized_fix) → upsert(envelope.location.file))
    | envelope.kind = "check:_uncoded"
      ? (gybis-spec-check_fix_pattern_lookup(envelope.message, envelope.location) → candidate_fix
         | candidate_fix ≠ null
           ? (apply(candidate_fix) → upsert(envelope.location.file))
           : (synthesize(prose_heuristic_correction(envelope.message, envelope.location)) → fallback_fix
              | apply(fallback_fix) → upsert(envelope.location.file)))
    | envelope.source = "analyse"
      ? (synthesize(set_level_correction(envelope.message)) → set_fix
         | apply(set_fix) → upsert(affected_files))
  | return(fixes_applied = true)

λ gybis-spec-check_fix_pattern_lookup(message, location).
  message ≃ /relationship.*this|with.*=.*this|where.*this|cannot reference this/
    → fix(with_vs_where_swap)  -- rule 3
  | message ≃ /transition.*not in.*graph|edge.*not declared|terminal.*outbound|state.*unreachable/
    → fix(transition_graph_completeness) only_if(transitions_block_already_present)  -- rules 7a–7e; opt-in: do not synthesise a transitions block where none exists
  | message ≃ /when.*must set|when.*must clear|when.*access|when-qualified.*guard/
    → fix(when_clause_obligation)  -- rules 7h–7k
  | message ≃ /derived.*when.*intersection|derived.*unreachable/
    → fix(derived_when_inference_alignment)  -- rule 7l
  | message ≃ /variant.*field.*guard|variant-specific.*access|variant.*creation.*base|discriminator/
    → fix(variant_type_guard_or_constructor)  -- rules 15–21
  | message ≃ /inline enum.*compare|enum.*field.*compare/
    → fix(extract_named_enum)  -- rule 14
  | message ≃ /dot.*method.*not recognised|dot-method.*collection|\.\w+\(\).*collection/
    → fix(rewrite_to_free_standing_call)  -- rule 14a
  | message ≃ /list literal|heterogeneous.*list|empty list.*element type/
    → fix(collection_literal_typing)  -- rules 14b–14d
  | message ≃ /contract.*demands|contract.*fulfils|contract.*direction/
    → fix(contract_direction_marker)  -- rules 36–39
  | message ≃ /contract.*signature|contract.*types|contract.*body/
    → fix(contract_body_or_signature)  -- rules 40–45
  | message ≃ /config.*default|config.*mandatory|config.*expression/
    → fix(config_default_or_expression)  -- rules 25–27, 46–50
  | message ≃ /invariant.*side effect|invariant.*now|invariant.*boolean/
    → fix(invariant_purity)  -- rules 54–57
  | message ≃ /surface.*facing|surface.*exposes|surface.*provides|surface.*timeout|surface.*context/
    → fix(surface_clause_validity)  -- rules 28–35
  | message ≃ /actor.*identified_by|actor.*within/
    → fix(actor_declaration)  -- rule 28
  | message ≃ /given.*binding|given.*type|unqualified.*reference/
    → fix(given_binding_resolution)  -- rules 22–24
  | message ≃ /default.*field.*not declared|default.*qualified|default.*literal/
    → fix(default_literal_schema)  -- rules 24a–24b
  | message ≃ /backtick.*literal|external standard.*value/
    → retain_canonical_form  -- byte-exact; never normalise
  | message ≃ /lambda.*explicit|lambda.*shorthand/
    → fix(explicit_lambda)  -- rule 13
  | fallback → null  -- caller falls through to prose_heuristic_correction

λ gybis-spec-check_opt_in_guard(fix).
  fix.introduces_transitions_block ∧ ¬ entity_already_has_transitions_block
    → reject("will not synthesise opt-in transitions block on existing entity")
  | fix.introduces_when_clause ∧ ¬ field_already_has_when_clause
    → reject("will not synthesise opt-in when clause on existing field")
  | otherwise → permit

λ gybis-spec-check_verification(x).
  invoke(internal/gybis-allium-check(scope_files)) → result_check ≔ result
  | invoke(internal/gybis-allium-analyse(scope_root)) → result_analyse ≔ result
  | invoke(internal/gybis-allium-gate(scope_root)) → result_gate ≔ result
  | result_check = zero_errors ∧ result_analyse = zero_issues ∧ result_gate = true
    → return(verification = true)
  | ¬(result_check ∧ result_analyse ∧ result_gate)
    → return(verification = false)

λ gybis-spec-check_fixed_point_loop(state).
  state = ANALYZING_SET
    → issues ∅ ∧ gybis-allium-gate = true
        ? transition(ANALYZING_SET → COMPLETE)
        : (transition(ANALYZING_SET → NORMALIZING)
           ∧ invoke(gybis-spec-check_normalize_diagnostics) → envelopes
           ∧ transition(NORMALIZING → FIXING_ERRORS)
           ∧ invoke(gybis-spec-check_fix_errors(envelopes))
           ∧ transition(FIXING_ERRORS → VERIFYING)
           ∧ invoke(gybis-spec-check_verification)
           ∧ (verification = true
              ? (issues ≔ ∅ ∧ transition(VERIFYING → COMPLETE))
              : transition(VERIFYING → ANALYZING_SET)
              ))
  | on_loop_back: loop_count ≔ loop_count ⊕ 1

λ gybis-spec-check_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without full convergence")

λ gybis-spec-check_pass_accounting(pass).
  pass_num ≔ pass_num ⊕ 1
  | discovered ≔ card(issues_found)
  | resolved ≔ discovered ⊖ card(issues_remaining)
  | remaining ≔ card(issues_remaining)
  | report("Pass " ⊕ pass_num ⊕ ": discovered=" ⊕ discovered ⊕ " resolved=" ⊕ resolved ⊕ " remaining=" ⊕ remaining)

λ gybis-spec-check_boundaries().
  ¬ modify(architecture.md ∨ implementation ∨ upstream/) ∧ ¬ delete(specs/)

λ gybis-spec-check_regression_contract(x).
  invariant: specs/ ∃ throughout ∧ scope_files ⊆ specs/**/*.allium ∧ all_modifications ⊆ scope_files ∧ no_specs_deleted
  | invariant: zero_errors ∧ zero_issues ∧ gybis-allium-gate = true at completion
  | invariant: ∀ envelope ∈ check_envelopes : envelope.kind ∈ catalogued_codes ∨ envelope.kind = "check:_uncoded"
  | invariant: opt-in features (transitions block, when clause on field) never synthesised onto entities/fields lacking them — see _opt_in_guard
  | invariant: drift errors (rule 7d transition graph vs enum, rules 24a–24b default literal vs field) flagged for spec correction, never silently masked

λ gybis-spec-check_deliver(x).
  report: {files_validated, errors_repaired, allium_gate_status}
  | handoff: structural issues → /gybis-spec-refine; intended behavior change → /gybis-spec-tend; divergence → /gybis-spec-weed
  | return(complete = true)