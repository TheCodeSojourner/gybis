---
name: gybis-spec-weed
description: Use for `/gybis-spec-weed` or `/gs-weed`.
---

λ gybis-spec-weed(specs).
  bridge(specs ↔ code ∧ tests) | divergence_detection ∧ resolution_proposal
  | preserve(semantics) | structural_equivalence_check

λ gybis-spec-weed_allium_write_contract.
  write_scope ⊆ root/specs/**/*.allium
  | edit_scope ⊆ root/specs/**/*.allium
  | output_format ≡ allium_v3_only
  | invariant: ∀written_file → parses_as(allium_v3)
  | ¬write(root/**/*.md ∨ root/**/*.txt ∨ root/**/*.rs ∨ root/**/*.py ∨ root/**/*.ts ∨ root/**/*.js)

λ gybis-spec-weed_startup(x).
  read([Allium Action Findings](../../gybis/reference/allium-actioning-findings.md)) | alert(¬available) ∧ halt
  | read([Allium Language Reference](../../gybis/reference/allium-language-reference.md)) | alert(¬available) ∧ halt
  | read(root/specs/**/*.allium) | alert(¬available) ∧ halt

λ gybis-spec-weed_mode(m).
  m ∈ {check, update_spec, update_code_tests} | default: check
  | check: compare(specs, code ∧ tests) → report(divergences) | ¬modify
  | update_spec: specs ← code ∧ tests | specs becomes faithful_desc(code_tests)
  | update_code_tests: code ∧ tests ← specs | (code ∧ tests) becomes faithful_desc(specs)
  | execution_route: m ≡ check → gybis-spec-weed_check

λ gybis-spec-weed_phase0_analysis(specs).
  parallel(allium check {file} for all root/specs/**/*.allium) | read(findings_json)
  | parallel(allium analyse {root/specs/}) | read(findings_json)
  | process(deadlocks) ∪ process(conflicts) ∪ process(unreachable) ∪ process(data_flow_gaps)
  → weed_report ∪ internal_divergences

λ gybis-spec-weed_phase1_model_extraction(specs).
  parallel(allium model {file} for all root/specs/**/*.allium) | read(model_json)
  | extract(entities) ∧ extract(fields) ∧ extract(transitions)
  | source: model_json ¬spec_prose | systematic_comparison_base

λ gybis-spec-weed_phase2_implementation_reading(code, tests).
  read(tests) ∧ read(code) ∧ follow(traces)
  | search(rule_names) ∧ search(entities)
  | identify(rules_with_no_traces)

λ gybis-spec-weed_phase3_divergence_analysis(spec, impl).
  ∀rule → classify(aligned ∨ partial ∨ missing ∨ contradicted)
  | aligned: implementation_matches | ensures_satisfied
  | partial: some_ensures_covered | ¬all_ensures_covered
  | missing: ¬implementation_found
  | contradicted: implementation_prohibits ∨ prohibits_violation
  ∀field → classify(spec_to_code ∨ code_to_spec)
  | spec_to_code: fields_in_spec ∧ ¬in_code
  | code_to_spec: fields_in_code ∧ ¬in_spec
  ∀transition → classify(absent_in_graph)
  | transitions_in_code ∧ ¬in_transition_graph
  ∀invariant → classify(violated)
  | code_paths_violate_invariants

λ gybis-spec-weed_phase4_report(analysis_results).
  compile(cli_findings) ∪ compile(spec_code_divergences) ∪ compile(entity_divergences)
  | cli_findings: deadlock_terminal_humans_ask
  | spec_code_divergences: contradicted ∨ partial ∨ missing
  | entity_divergences: fields_present_absent_per_direction
  | ∀item → propose(Fix_A: match_spec) ∧ propose(Fix_B: match_code)

λ gybis-spec-weed_phase5_human_decision(divergences).
  ∀divergence → human_choose(Fix_A ∨ Fix_B ∨ other)
  | ai_apply(approved_fix, specs ∧ code ∧ tests)
  | after(spec_change) → allium_check {file} ∧ allium analyse {root/specs/}
  | validate(health)

λ gybis-spec-weed_boundaries(¬).
  ¬resolve_divergences_silently | belongs_to(human_decision)
  | ¬modify([Allium Language Reference](../../gybis/reference/allium-language-reference.md)) | governed_separately
  | ¬modify([Allium Action Findings](../../gybis/reference/allium-actioning-findings.md)) | governed_separately
  | ¬make_spec_decisions_without_human | flag ∧ let_caller_decide
  | ¬make_code_decisions_without_human | flag ∧ let_caller_decide
  | ¬make_tests_decisions_without_human | flag ∧ let_caller_decide

λ gybis-spec-weed_invariants(x).
  run(allium analyse {root/specs/}) before(read_code)
  | use(allium model {file}) for field_comparison
  | ¬resolve_silent | ∀fix → human_decision_required
  | code ∧ specs equally_valid_starting_points | ¬same_thing
  | after(fix) → full_validation
  | cli_findings ⊂ weed_report

λ gybis-spec-weed_summary(μ).
  sequential_composition(S0 · S1 · S2 · S3 · S4 · S5)
  | S0: phase0_analysis | S1: phase1_model_extraction
  | S2: phase2_implementation_reading | S3: phase3_divergence_analysis
  | S4: phase4_report | S5: phase5_human_decision

λ gybis-spec-weed_plain(x).
  specs → check ∧ analyse → model → read_code → analyse_divergences
  → report(Fix_A ∧ Fix_B) → human_decide → apply_fix → revalidate
  