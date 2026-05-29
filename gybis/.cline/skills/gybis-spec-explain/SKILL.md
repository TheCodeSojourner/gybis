---
name: gybis-spec-explain
description: Use for `/gybis-spec-explain` or `/gs-explain`.
---

λ gybis-spec-explain(input_type, input_args).
  purpose(transform allium spec → plain-language prose) | target: developer_understanding ¬ parser_raw_syntax
  | input ∈ {domain_concern, domain, all_specs}
  | domain_concern: specs/{domain}/{concern}.allium
  | domain: specs/{domain}/**/*.allium
  | all_specs: specs/**.allium | "all" ∨ "all specs" → all

λ gybis-spec-explain_resolution(args).
  domain ∧ concern → read(specs/{domain}/{concern}.allium)
  | domain ∧ ¬concern → read(specs/{domain}/*.allium)
  | ¬domain ∧ ¬concern → read(specs/**/*allium)
  | "all" ∨ "all specs" → read(specs/**/*allium)
  | ambiguous → ask(user) ∧ halt ¬until_clarified

λ gybis-spec-explain_prose_generation(specs).
  produce(what_exists) ∧ produce(what_users_do) ∧ produce(rules) ∧ produce(guarantees) ∧ produce(domain_connections)
  | what_exists: purpose ∧ context ∧ key_concepts
  | what_users_do: actions ∧ outcomes ∧ triggers ∧ user_experience
  | rules: constraints ∧ prevention_mechanisms ∧ requirements ∧ edge_cases
  | guarantees: promises ∧ invariants ∧ terminal_states
  | domain_connections: user_journey_flow ∧ system_relationships

λ gybis-spec-explain_output_format(prose).
  flowing developer_friendly prose
  | light_headings for structure
  | ¬implementation_details | explanation ¬ design_document

λ gybis-spec-explain_quality_guarantees(output, specs).
  completeness: ∀ user_visible_behavior(specs) → represented(prose) | ¬omit_detail
  | cohesion: multi_file_specs → unified_narrative ¬ disjoint_excerpts
  | gap_surfacing: ambiguous ∧ gap ∧ unclear(specs) → explicit_questions(human)
  | read_only: ¬modify(specs) | only explain
  | explicit_non_application: ¬verifiable_guarantee → explicitly_stated ¬ silently_skipped

λ gybis-spec-explain_pipeline(input).
  resolve(input) → generate_prose → format_output → quality_check
  | quality_check(∀ guarantee_holds) → complete ¬until_pass
  