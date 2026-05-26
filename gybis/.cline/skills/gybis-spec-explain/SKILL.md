---
name: gybis-spec-explain
description: Use for `/gybis-spec-explain` or `/gs-explain`.
---

λ gybis_spec_explain(specs).
  purpose(transform(specs → developer_language_prose))
  | reference(../../gybis/reference/allium-language-reference.md)
  λ gybis_input_resolve(domain?, name?).
    specific(domain, name) → read(<root>/specs/{domain}/{name}.allium)
    | domain_only(domain) → read(<root>/specs/{domain}/*.allium)
    | unspecified() → read(<root>/specs/*/*.allium)
    | ambiguous() → ask_user(clarify)
  λ gybis_prose_generation(specs).
    produce(
      what_exists(purpose ∧ key_concepts) ∧
      what_users_can_do(actions ∧ outcomes ∧ triggers ∧ ux) ∧
      rules(constraints ∧ prevention ∧ requirements ∧ edge_cases) ∧
      guarantees(promises ∧ invariants ∧ terminal_states) ∧
      domain_connections(user_journey ∧ relationships)
    )
  λ gybis_output_format(prose).
    flowing(developer_friendly) ∧ light_headings ∧ ¬implementation_details
  λ gybis_quality_guarantees(output, specs).
    ∀user_visible_behavior(output) → represented ∧ ¬important_omission
    | multi_file → coherent_unified_narrative
    | gaps → surface_as_questions(human)
    | mode: read_only
    | ¬to_explain → explicit_say_so
  λ gybis_spec_explain_pipeline(specs).
    specs → gybis_input_resolve → gybis_prose_generation → gybis_output_format → output
    | quality_check(output, specs) ≡ ∀guarantee