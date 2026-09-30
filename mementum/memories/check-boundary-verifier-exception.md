---
type: Decision
symbol: 🎯
title: check-boundary-verifier-exception
related: arch-check-integrity-boundary, interaction-mode-taxonomy, requirements-layer-family
---

Rule: `check` diagnoses; `refine`, `tend`, and `weed` act. `spec-check` is the sole exception — it may also repair.

Why the exception is principled and not drift:
- `spec-check` has a deterministic external oracle: it fixes, then re-invokes `allium check` / `allium analyse` through `gybis-allium-gate`. Its `fixed_point_loop` cannot reach COMPLETE unless the tool agrees.
- `arch-check`, `req-check`, and `vocab-check` have no equivalent verifier. Their findings are semantic judgements (VSM coherence, requirement coverage, term completeness), so autonomous repair would mean the model deciding semantic questions unattended and rewriting the artifact.

Generalized rule: autonomous correction requires `verifier(external ∧ deterministic)`. Absent an oracle, a check must stay read-only and hand off to `refine`/`tend`/`weed`.

Rejected alternative: making all four checks repair autonomously (the "consistency by promoting the outlier" direction). It would have unified the family while removing the human gate for semantic change and duplicating the `refine`/`tend`/`weed` skill set.

Naming consequence: `check` still carries two behaviors, so `spec-check`'s purpose block and the check-family docs state the exception explicitly rather than leaving it implicit.
