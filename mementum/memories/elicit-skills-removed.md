---
type: Decision
symbol: 🎯
title: elicit-skills-removed
related: requirements-layer-family, human-owned-stage-readiness
---

`gybis-vocab-elicit` and `gybis-arch-elicit` were removed (session-42) at human direction.

Key decisions:

- Rationale: both were greenfield interview skills (`gate: artifact ¬∃`) that created `vocabulary.md`/`architecture.md` with no requirements grounding — a layer-ordering inversion once requirements became the top layer (req > vocab > arch > spec > tests > code).
- Precedent: `/gybis-spec-elicit` was removed earlier the same way; removal with replacement coverage is an established pattern.
- Greenfield replacement: `vocabulary.md` and `architecture.md` are human-authored with AI assistance from requirements, then validated by `/gybis-vocab-check` and `/gybis-arch-check`. No creation skill exists for these layers — tend/check/distill all require the artifact to exist.
- Greenfield loop: `/gybis-req-elicit` → human-authored vocab/arch → checks → `/gybis-arch-propagate` → `/gybis-spec-propagate`.
- References scrubbed: arch-describe/arch-explain halt pointers (now `/gybis-arch-distill` only), spec-propagate `S1_source_of_truth` (lambda-arch-distill.md only), allium-recommended-loops no_spec entry path, and all three doc surfaces (README, GYBIS-README, gybis-help).
- Lesson: when replacing table rows in gybis-help, verify the replacement row does not already exist elsewhere in the table — blind row swaps created duplicates that needed a second pass.