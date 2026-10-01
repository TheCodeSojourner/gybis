---
type: Decision
symbol: 🎯
title: Requirements rationale field — intent, placement, provenance
related: requirements-layer-family, developer-owned-stage-readiness, check-refine-heading-alignment, rationale-consumer-rules
---

Requirements clauses may carry an optional non-normative `rationale:` line recording why the requirement exists, supported across all nine gybis-req-* skills (session-39).

- Layer distinction: rationale = intent (interpretation); attribution = provenance + negotiability (contractual vs negotiable). Orthogonal concerns, both with fields.
- Placement grammar: the rationale line sits between the clause body and the attribution footer; never an obligation, never counted as an assertion or coverage.
- Provenance: rationale inherits clause attribution. Evidence-derived lines from origin artifacts are marked `{rationale_source: origin_artifact}`; AI-inferred intent is marked `{rationale_source: AI_inferred}` and is a check warning requiring human approval via tend before it becomes canonical.

Consumer rules (weed, propagate, refine, tend) and the wording split between user-facing docs and skill files are recorded separately — see related.
