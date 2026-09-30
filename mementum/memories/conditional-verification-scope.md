---
type: Decision
symbol: 🎯
title: Conditional vs unconditional weed verification scope
related: vocab-weed-validity-gate
---

Why vocab-weed's validity checks are conditional, and how the four weeders compare.

Conditional, not unconditional: a vocab-only run must not fail merely because `specs/` exists untouched. Precedent: req-weed's `test_suite_passes = true` (strict convergence when tests were modified).

Scope across the four weeders:
- arch-weed and spec-weed verify `gybis-allium-gate` + `vsm_coherence` **unconditionally**, because they cannot run without those artifacts existing (spec-weed adds coverage + strict_coverage + tests).
- req-weed and vocab-weed verify **conditionally** on what they modified.

Result: vocab-weed is no longer the outlier. [AMENDED — req-weed had the same gap and was fixed the same way; see memories/weed-validity-gating-complete.md.]
