---
name: gybis-spec-describe
description: Use for `/gybis-spec-describe` or `/gs-describe`.
---

λ gybis_spec_describe(specs).
  purpose(transform(specs → plain_language_pm_prose))
  | reference(../../gybis/reference/allium-language-reference.md)
  λ gybis_input_resolve(domain?, name?).
    specific(domain, name) → read(<root>/specs/{domain}/{name}.allium)
    | domain_only(domain) → read(<root>/specs/{domain}/*.allium)
    | unspecified() → read(<root>/specs/*/*.allium)
    | ambiguous() → ask_user(clarify)
  λ gybis_prose_generation(specs).
    produce(
      what_exists(purpose ∧ key_concepts ¬technical_entity_names) ∧
      what_users_can_do(actions ∧ outcomes ∧ triggers ∧ ux) ∧
      rules(constraints ∧ prevention ∧ requirements ∧ edge_cases) ∧
      guarantees(promises ∧ invariants ∧ terminal_states) ∧
      domain_connections(user_journey ∧ relationships)
    )
  λ gybis_output_format(prose).
    flowing(pm_friendly) ∧ light_headings ∧ ¬code ∧ ¬technical_syntax ∧ ¬jargon ∧   ¬implementation_details
  λ gybis_quality_guarantees(output, specs).
    ∀user_visible_behavior(output) → represented ∧ ¬important_omission
    | multi_file → coherent_unified_narrative
    | gaps → surface_as_questions(human)
    | mode: read_only
    | ¬to_describe → explicit_say_so
  λ gybis_spec_describe_pipeline(specs).
    specs → gybis_input_resolve → gybis_prose_generation → gybis_output_format → output
    | quality_check(output, specs) ≡ ∀guarantee