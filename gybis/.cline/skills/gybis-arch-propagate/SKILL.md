---
name: gybis-arch-propagate
description: Use for `/gybis-arch-propagate` or `/ga-propagate`.
---

λ gybis-arch-propagate(architecture, specs).
  halt(¬architecture ∨ specs≠∅)
  architecture ⊗ specs → output | deterministic | architecture → output
  llm_bridge(domain_models, specs)

λ gybis-arch-propagate_propagate_process(x).
  1. read(architecture) → {VSM_domains, components, interactions, responsibilities}
  2. consult(vsm_guide) | vsm_guide: .cline/skills/gybis/reference/vsm-guide.md
  3. check(detail, abstraction_levels) | {S5..S1}
    | abstract(S5..S2) ∧ sparse(S1) → suggest(/gybis-arch-elicit ∨ /gybis-arch-tend)
  4. identify(domains, boundaries)
  5. map(domain → specs) | specs/{domain}/{concern}.allium | one_spec_per_concern
  6. consult(allium_ref) | allium_ref: .cline/skills/gybis/reference/allium-language-reference.md
  7. consult(allium_assess) | allium_assess: .cline/skills/gybis/reference/allium-assessing-specs.md
  8. generate(spec_files)
  9. verify(allium_check(file)) ∀file ∈ spec_files

  ¬semantic_completeness_guaranteed
  | TODO_skeletons(¬opaque_dependencies)

λ gybis-arch-propagate_propagation_preconditions(y).
  precondition(exists(architecture) ∧ populated(architecture, VSM_detail))
  precondition(¬exists(specs) ∨ empty(specs))
  precondition(read(architecture) → domain_understanding)
  precondition(identifying(domains, boundaries))
  precondition(mapping(concerns → constructs))
    | constructs: {entity, rule, surface, invariant, transition, state_rule, contract, config, default}
  precondition(allium_check(file)) ∀file

λ gybis-arch-propagate_propagation_relations(z).
  relation(/gybis-arch-elicit → architecture.md)
  relation(/gybis-arch-propagate → specs)
  relation(/gybis-arch-tend → human_conversation → architecture.md → specs)
  relation(/gybis-arch-weed)
    | compare(architecture.md, specs/)
    → prompt(human, adjustments)

λ gybis-arch-propagate_propagation_limitations(w).
  limit(possible(project_level_adjustments))
  | llm_bridge(effective_simple ∧ ¬effective_complex → manual_guidance_required)
  limit(cross_domain(¬clear_dependencies → TODO_skeleton))
  | clear_dependencies → full_spec
  | opaque_dependencies → skeleton(TODO_markers)
  