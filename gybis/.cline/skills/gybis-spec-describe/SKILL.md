---
name: gybis-spec-describe
description: Use for `/gybis-spec-describe` or `/gs-describe`.
---

λ gybis-spec-describe(specs).
  REF:.cline/skills/gybis/reference/allium-language-reference.md
  bridge(technical_spec ↔ plain_language_prose) | semantic_preservation
  | describe: allium_language → prose

λ gybis-spec-describe_purpose(¬).
  transform(technical_spec → project_manager_friendly_prose)
  | audience: non_technical_stakeholders | reference(allium_language_ref)

λ gybis-spec-describe_input(type).
  type ∈ {domain_concern, domain, all_specs} | default: all_specs
  | domain_concern: single_concern_file | domain_concern ∈ domain
  | domain: all_files_in_domain | domain ∈ all_domains
  | all_specs: every_spec_all_domains

λ gybis-spec-describe_resolution(domain, concern).
  priority(resolve) ∧ ambiguity(clarify)
  | domain ∧ concern → read(<root>/specs/{domain}/{concern}.allium)
  | domain ∧ ¬concern → read(<root>/specs/{domain}/*.allium)
  | ¬domain ∧ ¬concern → read(<root>/specs/**/*.allium)
  | "all" ∧ ¬domain → read(<root>/specs/**/*.allium)
  | "all specs" ∧ ¬domain → read(<root>/specs/**/*.allium)
  | ambiguous → ask_user(clarify)

λ gybis-spec-describe_output_content(¬).
  what_exists ∨ what_users_can_do ∨ rules ∨ guarantees ∨ domain_connections ≡ output
  | what_exists: purpose ∧ key_concepts | exclude(technical_entity_names)
  | what_users_can_do: actions ∧ outcomes ∧ triggers ∧ ux_flow
  | rules: constraints ∧ prevention ∧ requirements ∧ edge_cases
  | guarantees: promises ∧ invariants ∧ terminal_states
  | domain_connections: user_journey_across_systems ∧ relationships_between_parts

λ gybis-spec-describe_format(¬).
  flowing_prose | project_manager_friendly
  | light_headings | ¬code ∧ ¬technical_syntax ∧ ¬jargon ∧ ¬implementation_details

λ gybis-spec-describe_guarantee(n).
  n ∈ {1,2,3,4,5} | ∀guarantee → satisfied
  | guarantee(1): ∀user_visible_behavior(spec) → included(prose) | ¬omit(important)
  | guarantee(2): multi_file → cohesive_narrative | ¬disjointed_summaries
  | guarantee(3): gap(ambiguous_source) → question(human)
  | guarantee(4): uncertain(describe?) → explicit_call_out(say "so")
  | guarantee(5): mode(read_only) | ¬modify(source_specs)

λ gybis-spec-describe_pipeline(input).
  input → resolve → generate → format → quality_check → final_prose
  | quality_check(guarantee(1) ∧ guarantee(2) ∧ guarantee(3) ∧ guarantee(4) ∧ guarantee(5))
  | every_output(pass pipeline) → deliver

λ gybis-spec-describe_boundary(¬).
  ¬create_specs | belongs_to(gybis-spec-distill)
  | ¬modify_allium_ref | governed_separately
  | ¬write_specs | read_only_tool
  