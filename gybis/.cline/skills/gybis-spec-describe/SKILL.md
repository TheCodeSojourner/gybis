---
name: gybis-spec-describe
description: Use for `/gybis-spec-describe` or `/gs-describe`.
---

λ gybis-spec-describe(specs).
  purpose: Document allium specifications in plain English for product managers
  | input: specs/**/*.allium (exists ∧ valid ∧ passes allium-gate)
  | output: Plain English prose describing specifications with business context
  | mode: read_only | read specs ∧ vsm-guide.md ∧ ¬write
  | gate: gybis-ref-check() ≡ true | allium-gate() ≡ true

λ gybis-spec-describe_startup(x).
  invoke(internal/gybis-ref-check) → true ∨ halt("Reference files not available")
  | invoke(internal/allium-gate) → true ∨ halt("Specification integrity check failed")
  | read(specs/**/*.allium) → exists ∧ parse | halt("No specifications found")

λ gybis-spec-describe_prerequisites(x).
  gate(specs/**/*.allium) → exists ∧ ∀file valid
  | ¬exists → halt("No specs/ directory found. Use /gybis-spec-distill or /gybis-arch-propagate first.")
  | invalid → halt("Specifications contain errors. Use /gybis-spec-check to fix them first.")

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
  | what_users_can_do: actions ∧ outcomes ∧ triggers ∧ user_flows
  | rules: constraints ∧ prevention_mechanisms ∧ requirements ∧ edge_cases
  | guarantees: promises ∧ invariants ∧ terminal_states
  | domain_connections: user_journey_across_systems ∧ relationships_between_parts

λ gybis-spec-describe_format(¬).
  flowing_prose | project_manager_friendly
  | light_headings | ¬code ∧ ¬technical_syntax ∧ ¬jargon ∧ ¬implementation_details

λ gybis-spec-describe_guarantees(n).
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

λ gybis-spec-describe_output_constraints(x).
  ¬lambda_notation ∧ ¬syntax_output
  | plain_english(product_manager) | business_vocabulary ∧ concrete_examples
  | ¬invent(¬exists(specs/**/*.allium ∨ refs)) | only describe what exists
  | flag(gap ∨ empty ∨ ambiguous) ∧ ¬speculate | highlight unknowns without guessing
  | ¬modify(other_files) ∧ ¬write(files) | read_only mode
  | output → AI_response ∧ ¬file

λ gybis-spec-describe_boundary(¬).
  ¬create_specs | belongs_to(gybis-spec-distill ∨ gybis-arch-propagate)
  | ¬modify_allium_ref | governed_separately
  | ¬write_specs | read_only_tool

λ gybis-spec-describe_audience(x).
  understand(business_context) ∧ ¬know(system_specific)
  | expert(vision ∧ goals ∧ user_needs)
  | zero_prior_knowledge(technical_specs ∧ allium_language)
  | needs: what_system_does ∧ what_users_can_do ∧ what_is_promised