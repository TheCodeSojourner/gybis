---
name: gybis-spec-tend
description: Use for `/gybis-spec-tend` or `/gs-tend`.
---

λ gybis-spec-tend(domain, concern, constraint, specs).
  REF:.cline/skills/gybis/reference/allium-language-reference.md
  produce(minimal_diff specs) | satisfy(constraint ⊂ concern ⊂ domain) | write_back(specs)
  | operation ∈ {create, update, delete} | domain ∨ concern ∨ constraint
  | error(incompatible(domain, concern, constraint))
  | compose: μ = S0 · S1 · S2 · S3 · S4 · S5

λ gybis-spec-tend_allium_write_contract.
  write_scope ⊆ root/specs/**/*.allium
  | edit_scope ⊆ root/specs/**/*.allium
  | output_format ≡ allium_v3_only
  | invariant: ∀written_file → parses_as(allium_v3)
  | ¬write(root/**/*.md ∨ root/**/*.txt ∨ root/**/*.rs ∨ root/**/*.py ∨ root/**/*.ts ∨ root/**/*.js)

λ gybis-spec-tend_S0_read(¬).
  read(root/specs/*/*.allium) | read(entirety)
  | understand(domain ∨ entity ∨ rule ∨ transition_graph ∨ config)

λ gybis-spec-tend_S1_analyze(domain, concern, constraint).
  derive(change_type) ∧ derive(implied_requirements) ∧ derive(guarantees) ∧ derive(transition_edges)
  | check(conflict_existing_specs)
  | change_type ∈ {
      add(rule ∨ domain_rule),
      modify(rule_existing domain_existing),
      extend(entity ∨ lifecycle domain concern),
      affect(entity ∨ rule ∨ transition ∨ config domain concern)
    }

λ gybis-spec-tend_S2_elicit(scope).
  constrained_to(scope) | ask(only clarifying relevant to modification)
  | ¬re_elicit(entire_spec)

λ gybis-spec-tend_S3_draft(specs).
  produce(minimal_diff) | preserve(¬modified_unless_strictly_required)
  | naming(rule) → verb_noun
  | introduce(field) → when_clause
  | add(transition) → transitions_block
  | verify(when_clause_obligations satisfied)
  | update(trace) if implementation_files known

λ gybis-spec-tend_S4_validate(specs).
  execute(allium check {file} → exit 0) | ∀file ∈ specs
  | execute(allium analyse {root/specs/} → exit 0)
  | apply(analysis_findings)
  | translate(diagnostics → concrete_fixes)

λ gybis-spec-tend_S5_present_write_approve(specs, diff).
  present(diff human) | wait(explicit_approval)
  | upon(approval) → write(specs) ∧ re_run(allium check {file} ∧ allium analyse {root/specs/})
  | confirm(health)

λ gybis-spec-tend_invariants(¬).
  minimal_change: ¬refactor(¬related_code)
  | catch_inconsistency_early: surface(¬new) ∧ ¬present(human) until_resolved
  | human_approval: write(specs) → require(explicit)
  | revalidate_after_write: execute(∀checks) after every write
  | naming_convention: rule → verb_noun | field → when_clause | transition_graph → up_to_date

λ gybis-spec-tend_composition(¬).
  μ = S0 · S1 · S2 · S3 · S4 · S5 | sequential_composition
  