---
type: Decision
symbol: 🎯
title: Check repair requires an oracle or explicit human authorization
related: arch-check-integrity-boundary, interaction-mode-taxonomy, requirements-layer-family
---

Rule: every check applies deterministic repairs automatically and prepares semantic repairs for approval. `spec-check` may repair autonomously under the Allium verifier; `req-check`, `arch-check`, and `vocab-check` use the same domain-local deterministic/approved-semantic split.

- `spec-check` re-runs `allium check` / `allium analyse` through `gybis-allium-gate`; it cannot complete unless that deterministic external oracle passes.
- `req-check` applies deterministic repairs automatically, then prepares semantic repairs in a bounded plan. Approval authorizes scoped repairs through owning skills; unresolved semantic conflicts stop the loop.
- `arch-check` and `vocab-check` apply deterministic repairs within their own artifacts and prepare bounded semantic repairs for approval.

Autonomous correction requires `verifier(external ∧ deterministic)`. Without one, repair needs explicit human authorization and bounded scope; otherwise checks hand off to `refine`/`tend`/`weed`.

Rejected: making every check repair autonomously would remove the semantic-change approval gate and duplicate `refine`/`tend`/`weed`.

Document both exceptions and their distinct gates; do not imply all checks behave alike.
