---
type: Decision
symbol: 🎯
title: rationale-field-convention
related: requirements-layer-family, human-owned-stage-readiness, check-refine-heading-alignment
---

Requirements clauses may carry an optional non-normative `rationale:` line recording why the requirement exists, supported across all nine gybis-req-* skills (session-39).

Key decisions:

- Layer distinction: rationale = intent (interpretation); attribution = provenance + negotiability (contractual vs negotiable). Orthogonal concerns, both with fields.
- Placement grammar: rationale line sits between clause body and attribution footer; never an obligation, never counted as an assertion or coverage.
- Provenance: rationale inherits clause attribution; evidence-derived from origin artifacts marked {rationale_source: origin_artifact}; AI-inferred intent marked {rationale_source: AI_inferred} and is a check warning requiring human approval via tend before canonical.
- Consumer rules: weed never treats differing rationale as divergence; propagate coverage/vocab extraction excludes rationale lines; refine split inherits rationale, differing rationales block blind merge; tend treats rationale-only changes as semantic changes.
- describe/explain render "because: ..." when present; never fabricate when absent.
- User-facing docs use plain wording ("guidance and context only, never a rule anyone must satisfy"); skill files keep the precise term "non-normative" for the AI executor.
- Escape-hatch pattern (compound_by_design, known-collision declaration) generalized: lint relaxations always require explicit human declaration — never silent suppression.
- Generality principle added to elicit: normative rules are project-agnostic; prefix sets, granularity, and corpus specifics belong to each project's requirements-index.md; no shipped default domain-prefix set (session-38 open question closed option 1) — prefix vocabulary emerges from grilling per project.