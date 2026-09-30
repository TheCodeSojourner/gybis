---
name: gybis-skill-contract-check
description: Internal skill - not user-facing
---

λ gybis-skill-contract-check(target = all).
  purpose: Verify each user-facing skill's declaration matches its contract for its kind
  | contract: pure_function(target → verdict) | ¬mutations | read_only
  | input: target ∈ {skill_name ∨ all}
  | output: true ∨ (false + violations)
  | gate: pre_condition | runs_on_demand ∨ runs_before(release)
  | rationale: six declared-vs-enforced divergences were found by hand; each was a value written once with no mechanism to detect its drift

λ gybis-skill-contract-check_required_fields(kind).
  domain → {purpose, input, output, interaction, gate} ∧ body ⊆ {state_machine, tool_guard, verify, boundary, boundaries, regression_contract, deliver}
  | memory → {purpose, input, output, interaction}
  | help → {output} ∧ ¬state_machine ∧ ¬tool_guard
  | delegate_required_iff: body = single_delegation | rationale: pure delegators declare delegate; skills with their own logic do not

λ gybis-skill-contract-check_expected_kind(skill_name).
  matches(name, gybis-(arch|req|spec|vocab)-*) → domain
  | matches(name, gybis-memory-*) ∨ name ∈ {gybis-fini, gybis-init} → memory
  | name = gybis-help → help

λ gybis-skill-contract-check_action(skill_name).
  suffix ≔ strip_prefix(name, gybis-(arch|req|spec|vocab)-)
  | suffix ∈ {check, refine, tend, weed, distill, propagate, describe, explain, elicit}

λ gybis-skill-contract-check_check_kind(skill).
  declared ≔ frontmatter(skill).kind
  | derived ≔ expected_kind(skill.name)
  | declared = derived ∨ violation(kind_mismatch, {declared, derived})

λ gybis-skill-contract-check_check_header(skill, kind).
  required ≔ required_fields(kind)
  | required ⊆ declared_fields(skill) ∨ violation(missing_fields, required ∖ declared_fields(skill))

λ gybis-skill-contract-check_check_interaction(skill).
  interaction ∈ {autonomous, interactive} ∨ violation(non_canonical_interaction, interaction)
  | mode_gate ∃ → admitted_modes ⊆ interaction ∨ violation(gate_admits_forbidden_mode, {admitted_modes, interaction})

λ gybis-skill-contract-check_check_loop_role(skill).
  _loop_role ∃
    → role ∈ canonical_roles(internal/reference/recommended-loops.md)
      ∧ read(internal/reference/recommended-loops.md) ∈ startup
    ∨ violation(undeclared_loop_role, {role, reference_read_present})

λ gybis-skill-contract-check_check_weed_verifier(skill).
  action(skill.name) = weed
    → ∀ artifact ∈ writable(skill.tool_guard): verifier ∃ for artifact
    ∨ violation(weeder_without_verifier, unverified_artifacts)

λ gybis-skill-contract-check_create_only(skill_name).
  action(skill_name) = distill → true | rationale: distill always creates a layer from the layers below
  | action(skill_name) = propagate ∧ suffix(skill_name) ∈ {arch, vocab} → true | rationale: these propagate create a new layer
  | action(skill_name) = propagate ∧ suffix(skill_name) ∈ {spec, req} → false | rationale: spec-propagate outputs regenerable code; req-propagate appends annotations
  | otherwise → false

λ gybis-skill-contract-check_absence_guard(artifact).
  -- canonical absence: negated existential `¬∃` or the empty-set operator `∅`
  ¬∃(artifact) ∨ artifact ¬∃ ∨ artifact ∅ ∨ ¬∃(glob(artifact))
  | rationale: SYSTEM_DESIGN.md operator table defines `∀`/`∃` (for all / there exists) ∧ `∅` (empty set / no result); the verbal predicate `exists(x)` is a gloss, not an operator

λ gybis-skill-contract-check_check_create_only(skill).
  create_only(skill.name)
    → gate contains absence_guard(own_output_artifact)
    ∨ violation(create_only_without_absence_guard, own_output_artifact)

λ gybis-skill-contract-check_check_absence_spelling(skill).
  -- verbal operator-form `exists`/`¬exists` is drift; identifier form (`X_exists`, `exists?`) ∧ prose ∧ `∅`-as-empty-set are exempt
  ∀ occurrence ∈ body(skill) where operator_form(occurrence) ∈ {exists, ¬exists}:
    violation(non_canonical_absence_spelling, occurrence)
  | ¬∃ such occurrence → true

λ gybis-skill-contract-check_check_reference_use(skill).
  reads_reference ≔ read_or_preload(skill, internal/reference/**) ∃
  | invokes_preflight ≔ invoke(internal/gybis-ref-check) ∃
  | invokes_preflight → reads_reference ∨ violation(reference_preflight_mismatch, {invokes_preflight, reads_reference})
  | rationale: many skills read reference files directly without the preflight; only the converse — a preflight with nothing to read — is a defect

λ gybis-skill-contract-check_check_boundaries(skill, kind).
  kind ≠ domain → true
  | -- describe/explain use _boundary() (singular, output-scoped) by design, not _boundaries()
    action(skill.name) ∈ {describe, explain} → _boundary() ∃ ∨ violation(output_boundary_missing, skill.name)
  | -- all other domain skills need both blocks, including read-only ones (arch-check ∧ vocab-check carry them)
    action(skill.name) ∉ {describe, explain} → _boundaries() ∃ ∧ _regression_contract() ∃
      ∨ violation(writing_skill_without_contract, {boundaries_present, regression_contract_present})

λ gybis-skill-contract-check_execution(target = all).
  skills ≔ target = all ? all_user_facing_skills() : {resolve(target)}
  | ∀ skill ∈ skills:
    kind ≔ frontmatter(skill).kind
    | collect(⋃ {
        check_kind(skill),
        check_header(skill, kind),
        check_interaction(skill),
        check_loop_role(skill),
        check_weed_verifier(skill),
        check_create_only(skill),
        check_absence_spelling(skill),
        check_reference_use(skill),
        check_boundaries(skill, kind)
      }) → violations
  | violations = ∅ → return(true)
  | violations ≠ ∅ → return(false ∧ violations)

λ gybis-skill-contract-check_regression_contract(x).
  invariant: read_only | ¬mutate(skill_files)
  | invariant: ∀ skill: kind ∈ {domain, memory, help}
  | invariant: ∀ violation: violation.kind ∈ catalogue
  | invariant: zero_violations → verification = true
