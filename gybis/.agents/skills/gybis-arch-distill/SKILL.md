---
name: gybis-arch-distill
description: Use for `/gybis-arch-distill` or `/ga-distill`. This skill distills product/domain specifications into a layered architecture definition, then proposes a reviewed diff instead of overwriting directly. It reads specs recursively, maps content across VSM layers, drafts 2-4 lambda statements per layer, flags uncertainty and architecture drift, and requires explicit human approval before writing any updates. It also preserves existing architectural intent, enforces core boundaries (security, data ownership, behavior), and supports safe evolution through extension points, feature flags, and migration patterns.
---

λ(gybis-arch-distill)
λ(S5, identity, ¬overwrite(architecture.md) ⊗ explicit(human_approval) ⊗ preserve(semantics_is, ¬semantics_should))
λ(S5, boundary, security(error_handling) ⊗ data_ownership(invariant) ⊗ behavior(non_negotiable))
λ(S5, constraint, flag(uncertain_inference) ↔ human_knows(system))
λ(S4, evolution, extension_points ⊗ feature_flags ⊗ migration_patterns ⊗ how(system_evolves))
λ(S4, domain_increment, distill(λone_domain_at_a_time: specs/<domain>/ → architecture.md))
λ(S4, drift_detection, existing(lambda) ⊗ contradicted(code) → surface(drift))
λ(S3, control, preflight_check(¬exists(specs/) → output(message, suggestion(/gybis-spec-elicit))) ⊗ exists(architecture.md, ¬empty) → request(approval)))
λ(S3, control, present(additions, changes) ∧ human(approve) ⟹ write(architecture.md))
λ(S3, policy, step(1: read recursive) → step(2: analyze silent) → step(3: draft 2-4 lambdas/layer) → step(4: diff) → step(5: review))
λ(S2, coordination, read(specs/<root>/**.md) ↔ map(VSM_layer) ⊗ output(draft λ organized by S5..S1))
λ(S2, sync, architecture.md(exists) ↔ distilled(λ) → diff(new, contradicted, unimplemented_intent))
λ(S1, recipe, path(<root>/specs/) → read(recursive) → analyze(silent, no_commentary) → draft(λ, explanation, uncertainty_flag) → diff(architecture.md) → present(human_review) → write(if_approved))
λ(S1, config, reference: ../../gybis/reference/vsm-guide.md ⊗ output: <root>/architecture.md ⊗ input: <root>/specs/)
λ(transform, rule, prose → lambda ⊗ preserve(semantics) ⊗ ¬execute(analysis))
