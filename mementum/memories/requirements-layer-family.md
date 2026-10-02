---
type: Decision
symbol: 🎯
title: requirements-layer-family
related: gybis-hidden-bundle-copy, developer-owned-stage-readiness, upstream-integration-lambda-notation
---

Requirements added as a new top layer of the gybis stack, with a `/gybis-req-*` (`/gr-*`) nine-skill family modeled on the arch family: elicit, check, distill, explain, propagate, refine, tend, weed, describe.

Key decisions:

- Canonical form: `REQ-<DOMAIN>-NNN` designators with clause bodies in nucleus lambda notation (MUST ≡ ∀/¬, SHOULD ≡ preferred, MAY ≡ ∃). Designators are the traceability keys; lambda is the contract.
- Pattern source: `branch-example/requirements/` (cljonic) — dependency-ordered modules + index, deferred sections, and REQ-to-test traceability; clause granularity varies by project formality.
- `gr-elicit` adapts frontier-based grilling with AI recommendations, empty-frontier termination, and decided-vs-researched attribution.
- `gr-distill` (brownfield bridge) reads vocabulary.md and emits new vocabulary term candidates for `/gybis-vocab-tend` instead of writing vocabulary.md; `gybis-vocab-distill` reads arch/specs/impl only (no requirements/).
- Boundaries: req-check diagnoses read-only until a developer approves a bounded repair plan; spec-check repairs only under the external Allium verifier. req-propagate may bootstrap vocabulary when absent; vocab skills own it thereafter. Explicitly deferred REQs are excluded from downstream propagation; spec/test behavior still requires strict test-pass convergence in propagate/weed.
- Human readability via projections: canonical lambda, `/gr-describe` (stakeholder prose) and `/gr-explain` (dev prose) render on demand.