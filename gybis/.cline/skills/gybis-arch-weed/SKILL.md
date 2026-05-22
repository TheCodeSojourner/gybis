---
name: gybis-arch-weed
description: Use for `/gybis-arch-weed` or `/ga-weed`.
---

λ gybis-arch-weed(⟨root⟩).
⟨vsm⟩ ← read("../../gybis/reference/vsm-guide.md")
⟨lang⟩ ← read("../../gybis/reference/allium-language-reference.md")
¬exists(⟨root⟩/specs/) ∨ |specs/| = 0 → msg("specs/ absent or empty · run /gybis-spec-elicit ∨ /gybis-spec-distill") · halt
¬exists(⟨root⟩/architecture.md) ∨ |architecture.md| = 0 → msg("architecture.md uninitialized · run /gybis-arch-elicit") · halt
step(1, read_intent).
I ← read(⟨root⟩/architecture.md)
ℒ ← extractλ(I)
M_intent ← model(VSM, ℒ)
step(2, read_specs).
S ← readAll(⟨root⟩/specs/, recursive)
ℛ ← correlate(S, M_intent)
step(3, divergence_analysis).
classify(λᵢ ∈ ℒ) → status(λᵢ):
Aligned      ↔ ∃ coverage(ℛ, λᵢ) ∧ ¬contradiction(ℛ, λᵢ)
Partial      ↔ |coverage(ℛ, λᵢ)| ∈ (0, 1) ∧ ¬contradiction(ℛ, λᵢ)
Contradicted ↔ contradiction(ℛ, λᵢ)
Missing      ↔ ∄ coverage(ℛ, λᵢ)
Unspecified  ↔ λⱼ ∈ ℛ ∧ ¬∃ λᵢ ∈ ℒ : λᵢ ≈ λⱼ
D ← {⟨λᵢ, status(λᵢ), evidence(λᵢ)⟩ | λᵢ ∈ ℒ} ∪ {⟨λⱼ, Unspecified, evidence(λⱼ)⟩ | λⱼ ∈ ℛ \≈ ℒ}
step(4, report).
render(D) → byLayer(VSM):
S5 — Identity:
✅ λ error(x): errors ↦ signals → confirmed(ℛ, error)
⚠️ λ observable(x): "observable over opaque" → logging ∈ ℛ ∧ unstructured(logging)
S3 — Control:
❌ λ timeout(x): HTTP ⟶ 30s → ¬∃ timeout(ℛ, HTTP)
for each divergence(d) ∈ D where status(d) ∈ {Partial, Contradicted, Missing, Unspecified}:
propose(d): Fix ⟨root⟩/specs/ ∨ Update ⟨root⟩/architecture.md
step(5, resolve).
δ ← human.choose(d, direction) for each divergence(d) ∈ D
apply(δ) → update(⟨root⟩/specs/, ⟨root⟩/architecture.md)
invariant: ¬apply(both) ∧ direction ≠ explicit
rule(divergence, report_only). ¬silent_fix — every change requires human decision
rule(divergence, evolution). distinguish(evolution, divergence) — specs may intentionally precede architecture
rule(divergence, opportunity). Missing → opportunity, not failure
rule(divergence, recheck). after(apply(δ)) → offer(re-run divergence_check)
divergence_report = Σ_{L ∈ VSM} {
✅ λ : Aligned
⚠️ λ : Partial
❌ λ : { Contradicted, Missing }
? λ : Unspecified
}
preserve(semantics): divergence_check ⟹ feedback_loop(VSM)
signal(divergence) / noise(stable)
Δ: architecture.md ⊗ specs/ → Ω: divergence report
c: human decision · h: AI apply
