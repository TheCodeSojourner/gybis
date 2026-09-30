---
type: Decision
symbol: 🎯
title: X-propagate creates the layer below X; req-propagate gained vocab write authority
related: propagate-seed-then-own
---

Convention: `X-propagate` creates the layer immediately below `X`.

| Skill             | Creates                    |
| ----------------- | -------------------------- |
| `req-propagate`   | req → vocabulary           |
| `vocab-propagate` | vocab → architecture (new) |
| `arch-propagate`  | arch → specs               |
| `spec-propagate`  | specs → code/tests         |

This resolves the earlier anomaly where `req-propagate` created no layer; it needed write authority over `vocabulary.md`.

Boundary change: `req-propagate` previously carried `constraint: ¬mutate(vocabulary.md)` and could only emit candidates. It now writes `vocabulary.md` when absent (seeded `status: draft`, `seeded_from: requirements/`, unresolved synonym conflicts recorded as open questions) and appends terms only when present. The invariant narrows to "vocab skills own `vocabulary.md` after bootstrap".

`vocab-propagate` design: reads **both** `requirements/` and `vocabulary.md` — req drives architectural content, vocabulary drives terminology. Gate `requirements/ ∃ ∧ vocabulary.md ∃ ∧ architecture.md ¬∃`; `interaction: autonomous`; carries the canonical block set. Precedent for cumulative reads: `spec-propagate` already reads arch + specs.

Not changed: `arch-propagate` still reads `architecture.md` only — terms are baked in by vocab-propagate, so specs inherit them transitively.
