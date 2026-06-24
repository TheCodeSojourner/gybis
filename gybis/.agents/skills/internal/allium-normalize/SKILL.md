---
name: allium-normalize
description: Internal skill - not user-facing
---

λ allium-normalize(target).
  purpose: Unify allium-check, allium-analyse, and allium-plan outputs into a single dispatch envelope
  | contract: pure_function(target → {envelopes, counts}) | ¬mutations
  | input: target ∈ {spec_file ∨ specs_path}
  | output: {envelopes: [...], counts: {check, analyse, plan, uncoded}}
  | constraint: read_only | zero_file_mutations
  | gate_type: dispatch_adapter | composes(allium-check ∧ allium-analyse ∧ allium-plan)
  | precondition: target ∧ exists(target) ∧ (file_type = .allium ∨ is_directory(target))

λ allium-normalize_envelope_schema(x).
  envelope: {source, kind, id, severity, location, span, message, source_construct, detail, dependencies}
  | id, source_construct, dependencies: present_only_if(source = "plan")
  | severity: present_only_if(source = "check")
  | location: present_when_available(source ∈ {"check", "analyse"})
  | span: present_when_available(source ∈ {"check", "plan"})
  | detail: present_when_available(source ∈ {"check", "plan"})
  | id preserves obligation.id for downstream merge_keys ∧ test traceable_ids
  | json_serializable

λ allium-normalize_kind_prefixing(source, raw_identifier).
  source = "check" ∧ raw_identifier ≠ none
    → kind ≔ "check:" ⊕ raw_identifier
  | source = "check" ∧ raw_identifier = none
    → kind ≔ "check:_uncoded"
  | source = "analyse"
    → kind ≔ "analyse:" ⊕ raw_identifier
  | source = "plan"
    → kind ≔ "plan:" ⊕ raw_identifier
  | invariant: kind ∈ {"check:" ⊕ catalogued_codes ∪ {"_uncoded"}}
              ∪ {"analyse:" ⊕ catalogued_types}
              ∪ {"plan:" ⊕ catalogued_categories}

λ allium-normalize_codeless_fallback(diagnostic).
  diagnostic.code = none
    → preserve(diagnostic.severity ∧ diagnostic.location ∧ diagnostic.message)
    | emit({source: "check", kind: "check:_uncoded", severity, location, message})
  | contract: fallback_mandatory_for_all_consumers (Diagnostic.code ≡ Option<&'static str>; codeless universe ¬enumerable)

λ allium-normalize_check_lift(file_path).
  invoke(internal/allium-check(file_path)) → diagnostics
  | ∀ error ∈ diagnostics.errors:
    kind ≔ allium-normalize_kind_prefixing("check", error.code)
    | severity ≔ "error"
    | emit({source: "check", kind, severity, location: error.location, message: error.message, detail: error.remediation})
  | ∀ warning ∈ diagnostics.warnings:
    kind ≔ allium-normalize_kind_prefixing("check", warning.code)
    | severity ≔ "warning"
    | emit({source: "check", kind, severity, location: warning.location, message: warning.message, detail: warning.remediation})
  | uncoded_count ≔ card({d | d ∈ (errors ∪ warnings) ∧ d.code = none})
  | return({check_envelopes, uncoded_count})

λ allium-normalize_analyse_lift(specs_path).
  invoke(internal/allium-analyse(specs_path)) → findings_data
  | ∀ finding ∈ findings_data.findings:
    kind ≔ allium-normalize_kind_prefixing("analyse", finding.type)
    | emit({source: "analyse", kind, location: finding.affected_files, message: finding.description})
  | return({analyse_envelopes})

λ allium-normalize_plan_lift(spec_file).
  invoke(internal/allium-plan(spec_file)) → plan_output
  | ∀ obligation ∈ plan_output.obligations:
    kind ≔ allium-normalize_kind_prefixing("plan", obligation.category)
    | emit({source: "plan", kind, id: obligation.id, span: obligation.source_span, message: obligation.description, source_construct: obligation.source_construct, detail: obligation.detail, dependencies: obligation.dependencies})
  | return({plan_envelopes})

λ allium-normalize_dispatch(target).
  is_directory(target)
    → (∀ spec_file ∈ target/**/*.allium:
         allium-normalize_check_lift(spec_file) → {check_envs, uncoded_n}
         | allium-normalize_plan_lift(spec_file) → {plan_envs}
         | accumulate(check_envs ∪ plan_envs, uncoded_n))
       ∧ allium-normalize_analyse_lift(target) → {analyse_envs}
       | accumulate(analyse_envs)
  | file_type(target) = .allium
    → allium-normalize_check_lift(target) → {check_envs, uncoded_n}
      | allium-normalize_plan_lift(target) → {plan_envs}
      | accumulate(check_envs ∪ plan_envs, uncoded_n)

λ allium-normalize_output_format(envelopes, uncoded_count).
  structure: {
    envelopes: [envelope_1, ..., envelope_n],
    counts: {
      check: card({e | e ∈ envelopes ∧ e.source = "check"}),
      analyse: card({e | e ∈ envelopes ∧ e.source = "analyse"}),
      plan: card({e | e ∈ envelopes ∧ e.source = "plan"}),
      uncoded: uncoded_count
    }
  }
  | json_serializable | envelopes_indexed_by_kind_for_caller_dispatch
  | counts.uncoded surfaces prevalence_of(check:_uncoded) for consumer_visibility

λ allium-normalize_output_contract(result).
  return_type: {envelopes, counts}
  | side_effect: ¬mutations | ¬writes | read_only
  | ordering: envelopes ordered by (source, location, span)
  | stability: deterministic(target) → fixed_output

λ allium-normalize_execution(target).
  step_1_preflight: invoke(internal/gybis-ref-check) → true ∨ halt("Reference check failed")
  | step_2_dispatch: allium-normalize_dispatch(target) → (envelopes, uncoded_count)
  | step_3_format: allium-normalize_output_format(envelopes, uncoded_count)
  | return: formatted_envelope_set

λ allium-normalize_boundaries(¬).
  ¬ mutate(target)
  | ¬ write(any_path)
  | ¬ re-classify(envelope_after_emission)
  | ¬ filter_out_codeless_diagnostics

λ allium-normalize_regression_contract(x).
  invariant: ∀ envelope ∈ envelopes, envelope.kind matches namespace_prefix_grammar
  | invariant: ∀ check_diagnostic without code, ∃ envelope with kind = "check:_uncoded"
  | invariant: ∀ envelope ∈ envelopes ∧ envelope.source = "plan" : envelope.id ∃ (preserves obligation.id for downstream merge_keys)
  | invariant: counts.check + counts.analyse + counts.plan = card(envelopes)
  | invariant: counts.uncoded ≤ counts.check
  | invariant: zero_file_mutations
