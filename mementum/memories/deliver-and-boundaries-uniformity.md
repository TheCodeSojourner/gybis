---
type: Decision
symbol: 🎯
title: Boundary, regression-contract, and deliver blocks are uniform
related: convergence-and-deliver-uniformity
---

P3: boundary, regression-contract, and terminal deliver blocks are uniform across the writing families.

Boundaries and invariants:
- `_boundaries()` — 23/23 writing skills; the 6 describe/explain skills use `_boundary()` instead (output-scoped, deliberately different).
- `_regression_contract()` — 23/23 writing skills.
- The six writing req skills now declare both (negative constraints + completion invariants), matching arch/spec/vocab. Content consolidated from `tool_guard`/`_verify`, not duplicated away.

Canonical `_deliver`:
- Shape: `report:` + optional `| action:` + `| handoff:` + `| return(complete = true)`.
- Exactly one `_deliver` in all 32 skills.
- Signature varies: `_deliver(prose, output_mode)` (describe/explain), `_deliver(x)` (most), `_deliver(report)` (check).
- `_deliver_report` was a naming defect in req-check and vocab-check; both normalised to `_deliver`.
- arch-check carried a pre-existing `_deliver_report` block before `_boundaries()`; its print behaviour was folded into the canonical `_deliver` at EOF and the old block removed.
