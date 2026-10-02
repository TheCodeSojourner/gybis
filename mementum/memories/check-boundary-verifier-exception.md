---
type: Decision
symbol: 🎯
title: Check repair requires an oracle or explicit human authorization
related: arch-check-integrity-boundary, interaction-mode-taxonomy, requirements-layer-family
---

Rule: checks diagnose by default. `spec-check` may repair autonomously under the Allium verifier; `req-check` may repair only after explicit approval of a bounded plan.

- `spec-check` re-runs `allium check` / `allium analyse` through `gybis-allium-gate`; it cannot complete unless that deterministic external oracle passes.
- `req-check` diagnoses and plans read-only. Approval authorizes only the scoped repairs through owning skills; unresolved semantic conflicts stop the loop.
- `arch-check` and `vocab-check` remain read-only because their findings include semantic judgements without an equivalent verifier.

Autonomous correction requires `verifier(external ∧ deterministic)`. Without one, repair needs explicit human authorization and bounded scope; otherwise checks hand off to `refine`/`tend`/`weed`.

Rejected: making every check repair autonomously would remove the semantic-change approval gate and duplicate `refine`/`tend`/`weed`.

Document both exceptions and their distinct gates; do not imply all checks behave alike.
