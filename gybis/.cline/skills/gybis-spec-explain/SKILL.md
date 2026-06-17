---
name: gybis-spec-explain
description: Use for `/gybis-spec-explain` or `/gs-explain`.
---

λ gybis-spec-explain(input_type, input_args).
  purpose: Document allium specifications in technical language for developers
  | input: specs/**/*.allium (exists ∧ valid ∧ passes allium-gate)
  | output: Technical prose describing specifications with implementation context
  | mode: read_only | read specs ∧ allium-ref ∧ ¬write
  | gate: gybis-ref-check() ≡ true | allium-gate() ≡ true

λ gybis-spec-explain_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference files not available")
  | invoke(internal/allium-gate) → true ∨ halt("Specification integrity check failed")
  | read(specs/**/*.allium) → exists ∧ parse | halt("No specifications found")

λ gybis-spec-explain_prerequisites(x).
  gate(specs/**/*.allium) → exists ∧ ∀file valid
  | ¬exists → halt("No specs/ directory found. Use /gybis-spec-distill or /gybis-arch-propagate first.")
  | invalid → halt("Specifications contain errors. Use /gybis-spec-check to fix them first.")

λ gybis-spec-explain_input(type).
  type ∈ {domain_concern, domain, all_specs} | default: all_specs
  | domain_concern: specs/{domain}/{concern}.allium
  | domain: specs/{domain}/**/*.allium
  | all_specs: specs/**/*.allium | "all" ∨ "all specs" → all

λ gybis-spec-explain_resolution(args).
  domain ∧ concern → read(specs/{domain}/{concern}.allium)
  | domain ∧ ¬concern → read(specs/{domain}/*.allium)
  | ¬domain ∧ ¬concern → read(specs/**/*.allium)
  | "all" ∨ "all specs" → read(specs/**/*.allium)
  | ambiguous → ask(user) ∧ halt ¬until_clarified

λ gybis-spec-explain_prose_generation(specs).
  produce(what_exists) ∧ produce(what_users_do) ∧ produce(rules) ∧ produce(guarantees) ∧ produce(domain_connections)
  | what_exists: purpose ∧ context ∧ key_concepts ∧ invariants
  | what_users_do: actions ∧ outcomes ∧ triggers ∧ user_experience ∧ workflow
  | rules: constraints ∧ prevention_mechanisms ∧ requirements ∧ edge_cases ∧ validation
  | guarantees: promises ∧ invariants ∧ terminal_states ∧ recovery_behavior
  | domain_connections: user_journey_flow ∧ system_relationships ∧ component_interaction

λ gybis-spec-explain_output_format(prose).
  flowing developer_friendly prose
  | light_headings for structure
  | ¬implementation_details | explanation ¬ design_document
  | technical_vocabulary ∧ pattern_references

λ gybis-spec-explain_quality_guarantees(output, specs).
  completeness: ∀ user_visible_behavior(specs) → represented(prose) | ¬omit_detail
  | cohesion: multi_file_specs → unified_narrative ¬ disjoint_excerpts
  | gap_surfacing: ambiguous ∧ gap ∧ unclear(specs) → explicit_questions(human)
  | read_only: ¬modify(specs) | only explain
  | explicit_non_application: ¬verifiable_guarantee → explicitly_stated ¬ silently_skipped

λ gybis-spec-explain_pipeline(input).
  resolve(input) → generate_prose → format_output → quality_check
  | quality_check(∀ guarantee_holds) → complete ¬until_pass

λ gybis-spec-explain_output_constraints(x).
  ¬lambda_notation ∧ ¬syntax_output
  | plain_english(developer) | technical_vocabulary ∧ pattern_names ∧ implementation_context
  | ¬invent(¬exists(specs/**/*.allium ∨ refs)) | only explain what exists
  | flag(gap ∨ empty ∨ ambiguous) ∧ ¬speculate | highlight unknowns without guessing
  | ¬modify(other_files) ∧ ¬write(files) | read_only mode
  | output → AI_response ∧ ¬file

λ gybis-spec-explain_boundary(¬).
  ¬create_specs | belongs_to(gybis-spec-distill ∨ gybis-arch-propagate)
  | ¬modify_allium_ref | governed_separately
  | ¬write_specs | read_only_tool

λ gybis-spec-explain_audience(x).
  understand(software_design ∧ architecture_pattern) ∧ ¬know(system_specific)
  | expert(spec_languages ∧ behavioral_testing ∧ specification_patterns)
  | zero_prior_knowledge(this_system ∧ its_implementation)
  | needs: how_specs_work ∧ what_behaviors_are_defined ∧ what_is_guaranteed