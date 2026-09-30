---
type: Decision
symbol: 🎯
title: authority-invocation-gate
related: describe-explain-output-modes, gybis-tool-agnostic
---

Running a gybis command is the human approval for the writes that command is defined to make.

Decision:
- `distill` and `propagate` write their artifacts autonomously once invoked; there is no per-write confirmation gate inside them.
- Review happens after the fact through `check` and `weed` (and `tend` for intended change).
- Only unrequested writes are prohibited; there are no covert or unsolicited writes.

Rationale:
- Clarifies the README Authority Model clauses "Every write operation requires human approval" and "No autonomous AI actions", which otherwise read as violated by design.
- Keeps the approval gate at the command boundary instead of multiplying prompts inside long-running transforms.

Consequences:
- `arch-propagate` / `spec-propagate` / `req-propagate` / `*-distill` remain autonomous-write skills.
- `weed` and `tend` remain interactive: they resolve divergence and evolve intent, which are decisions, not transforms.
