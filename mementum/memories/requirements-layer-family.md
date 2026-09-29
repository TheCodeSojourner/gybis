---
type: Decision
symbol: 🎯
title: requirements-layer-family
related: gybis-hidden-bundle-copy, human-owned-stage-readiness, upstream-integration-lambda-notation
---

Requirements added as a new top layer of the gybis stack, with a `/gybis-req-*` (`/gr-*`) nine-skill family modeled on the arch family: elicit, check, distill, explain, propagate, refine, tend, weed, describe.

Key decisions:

- Canonical form: `REQ-<DOMAIN>-NNN` designators with clause bodies in nucleus lambda notation (MUST ≡ ∀/¬, SHOULD ≡ preferred, MAY ≡ ∃). Designators are the traceability keys; lambda is the contract.
- Pattern source: `branch-example/requirements/` (cljonic) — dependency-ordered module files + index, deferred future-work sections, requirements-to-test traceability. Adopt the skeleton; make clause granularity configurable (library-contract vs application) so density matches project formality.
- `gr-elicit` is based on mattpocock/skills `grilling` protocol: frontier rounds with numbered questions and AI-recommended answers, empty-frontier termination, facts-researched-by-AI. Adapted with decided-vs-researched attribution so weed knows negotiable vs contractual conflicts.
- `gr-distill` (brownfield bridge) also emits vocabulary term candidates and hands them to `/gybis-vocab-tend` instead of writing vocabulary.md; `gybis-vocab-distill` now lists requirements/ as a source.
- Boundaries preserved: check is read-only diagnostics; vocabulary mutation stays in vocab skills; spec/test/behavior changes require strict test-pass convergence in propagate/weed.
- Human readability via projections: canonical lambda, `/gr-describe` (stakeholder prose) and `/gr-explain` (dev prose) render on demand.