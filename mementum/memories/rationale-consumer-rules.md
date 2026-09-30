---
type: Decision
symbol: 🎯
title: Requirements rationale — consumer rules and wording
related: rationale-field-convention
---

Consumer rules and wording for the requirements `rationale:` line.

- weed never treats a differing rationale as divergence.
- propagate coverage/vocab extraction excludes rationale lines.
- refine split inherits rationale; differing rationales block blind merge.
- tend treats rationale-only changes as semantic changes.
- describe/explain render "because: ..." when present; never fabricate when absent.
- User-facing docs use plain wording ("guidance and context only, never a rule anyone must satisfy"); skill files keep the precise term "non-normative" for the AI executor.

Escape-hatch pattern (`compound_by_design`, known-collision declaration) generalized: lint relaxations always require explicit human declaration — never silent suppression.

Generality principle added to elicit: normative rules are project-agnostic; prefix sets, granularity, and corpus specifics belong to each project's requirements-index.md. No shipped default domain-prefix set — prefix vocabulary emerges from grilling per project.
