---
type: Decision
symbol: 🎯
title: Loop-guard and pass-accounting uniformity
related: deliver-and-boundaries-uniformity, interaction-mode-taxonomy, requirements-layer-family
---

P3: loop guarding and pass accounting are uniform across the five skill families.

Loop guards — canonical mechanism (all looping skills): `| on_loop_back: loop_count ≔ loop_count ⊕ 1` at each loop-back, plus a guard lambda:
```
λ X_loop_guard(state).
  loop_count ≥ max_iterations
    → halt("Maximum iterations reached without full convergence")
```
- Only arch-distill and spec-distill extend the predicate with `∨ no_progress_detected` (strictly stronger; deliberate) and emit a `diagnostic_report`.
- Removed superseded forms: prose `loop_guard: iteration_count ≤ max_iterations`, the `condition:`/`action_on_trigger:` guard shape, and guards that incremented the counter inside the guard.
- Fixed: the six looping req skills had unbounded retry loops; arch-propagate/spec-propagate had unguarded `re_synthesize_*` loops; vocab-tend had an unbounded `VERIFY_CONSISTENCY → ELICIT_FEEDBACK`. Worst case was the autonomous skills, where no human watches.

Pass accounting: `_pass_accounting(pass)` present in all 21 looping skills (was 12). Order everywhere: `_loop_guard` → `_pass_accounting` → `_boundaries` → `_regression_contract`.
