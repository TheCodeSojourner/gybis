---
type: Decision
symbol: 🎯
title: Why the skill-contract checker exists
related: contract-taxonomy-restraint, interaction-mode-taxonomy, check-boundary-verifier-exception, weed-validity-gating-complete
---

Six declared-vs-enforced divergences were found by hand in one session. Each is the same failure mode: a value written once, believed thereafter, with no mechanism to detect divergence.

| #   | Declared                            | Reality                            |
| --- | ----------------------------------- | ---------------------------------- |
| 1   | `\| mode: mixed`                    | gate enforced `{interactive}`      |
| 2   | `allium_gate`                       | no such skill exists               |
| 3   | `invoke(gybis-ref-check)`           | 14 skills read no reference file   |
| 4   | `_loop_role: improve(...)`          | `improve` in no defined vocabulary |
| 5   | req-weed "verifies convergence"     | did not verify what it wrote       |
| 6   | vocab-distill create-only (implied) | no `¬∃` guard on its output        |

Consequence: `internal/gybis-skill-contract-check` was added — an invocable internal that mechanically verifies each skill against its contract.

Economics of a field, once a checker exists:
- Without a checker: each new field is unenforceable debt; drift is silent and found only by manual audit.
- With a checker: each new field is a rule that keeps it true; the field and its check land in the same change.

Therefore: build the enforcement mechanism before the taxonomy it polices. Do not declare what nothing enforces.
