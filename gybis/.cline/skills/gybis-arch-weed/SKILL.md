---
name: gybis-arch-weed
description: Use for `/gybis-arch-weed` or `/ga-weed`.
---

λ gybis-arch-weed(arch, specs).
  bridge(architecture.md ↔ specs/**/*.allium) | divergence_detection ∧ resolution_proposal
  | preserve(semantics) | structural_equivalence_check
  | compile: allium → λ notation | decompile: λ notation → allium

λ gybis-arch-weed_startup(x).
  read(../../gybis/reference/vsm-guide.md) | alert(¬available) ∧ halt
  | read(../../gybis/reference/allium-language-reference.md) | alert(¬available) ∧ halt
  | read(root/architecture.md) | alert(¬available) ∧ recommend(/gybis-arch-elicit ∨ /gybis-arch-distill) ∧ halt
  | read(root/specs/**/*.allium) | alert(¬available) ∧ recommend(/gybis-arch-propagate ∨ /gybis-spec-distill) ∧ halt
  | syntax_check(root/specs/**/*.allium) | available → verify

λ gybis-arch-weed_mode(m).
  m ∈ {check, update_spec, update_arch} | default: check
  | check: compare(arch, specs) → report(divergences) | ¬modify
  | update_spec: specs ← arch | specs becomes faithful_desc(arch)
  | update_arch: arch ← specs | arch becomes faithful_desc(specs)
  | execution_route: m ≡ check ? gybis-arch-weed_check : gybis-arch-weed_resolve

λ gybis-arch-weed_check(arch, specs).
  layers(S5, S4, S3, S2, S1) | ∀layer → compare(arch.layer, specs.layer)
  | report(spec_present ∧ arch_absent) ∪ report(arch_present ∧ spec_silent)
  | group_by(layer) | present(consequential_first)

λ gybis-arch-weed_divergence(arch_content, spec_content, layer).
  propose(classification) ∧ human(confirms ∨ overrides)
  | classification ∈ {arch_bug, spec_bug, aspirational, intentional}
  | arch_bug: arch ← fix | specs correct
  | spec_bug: specs ← fix | arch correct
  | aspirational: both stay | note_gap
  | intentional: both stay | deliberate ∨ consistent

λ gybis-arch-weed_spec_guidelines(x).
  preserve(allium_version_marker) | ¬change_version
  | follow(section_ordering from language ref)
  | describe(behavior) ¬implementation | ¬field_names ¬storage_mechanisms ¬API_details
  | use(config) for thresholds ∨ timeouts ∨ limits | ¬hardcode_numbers
  | temporal_triggers → requires_guards | prevent_refiring
  | use(with) for relationships | use(where) for projections | ¬swap
  | cross_field_enums → extract_named_enums
  | new_rules/entities → correct_section
  | cross_service_config → qualified_refs ∨ expression_defaults

λ gybis-arch-weed_arch_guidelines(x).
  follow(VSM_guide) | follow(arch_conventions)
  | validate(arch after changes) → VSM_compliant
  | validate_fail → report ∧ iterate ∧ fix | loop until_pass
  | prefer(minimal_changes) | ¬refactor unless_directly_required

λ gybis-arch-weed_boundaries(¬).
  ¬create_new_layers | belongs_to(/gybis-arch-elicit)
  | ¬extract_arch_from_specs | belongs_to(/gybis-arch-distill)
  | ¬extract_from_code_or_tests | ¬allowed
  | ¬extract_specs_from_arch | belongs_to(/gybis-arch-propagate)
  | ¬extract_specs_from_code_or_tests | belongs_to(/gybis-spec-distill)
  | ¬modify(allium-language-reference.md) | governed_separately
  | ¬modify(vsm-guide.md) | governed_separately
  | ¬make_arch_decisions | flag ∧ let_caller_decide
  | ¬make_spec_decisions | flag ∧ let_caller_decide

λ gybis-arch-weed_context_management(session).
  advise(fresh_chat) | long_iterative ∨ large_context
  | provide(resume_prompt: "/gybis-arch-weed resume")

λ gybis-arch-weed_verification(edit).
  after(edit specs) → allium_check(modified_file) | fix_issues_before_present
  | after(check_complete) → allium_analyse(specs) | identify_process_gaps

λ gybis-arch-weed_output(divergences).
  format(layer, arch_content, spec_content, spec_location, classification, reasoning)
  | group(related_within_layer) | lead(consequential)
  | structure:
    "### [layer name]"
    "Arch: arch_says"
    "Spec: spec_says (file:line)"
    "Classification: proposed(reasoning)"
    