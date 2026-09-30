---
type: Decision
symbol: 🎯
title: vocab-weed gates completion on validity of what it modified
related: check-boundary-verifier-exception, convergence-and-deliver-uniformity, requirements-layer-family, conditional-verification-scope
---

P4: vocab-weed rewrites `specs/`, `architecture.md` and implementation by blind global term replacement but verified only zero_divergences — so a replacement that broke Allium syntax, VSM coherence, or tests could still report COMPLETE.

Decision: gate completion on validity checks, conditional on what was actually modified.

Mechanism:
- `modified_artifacts ≔ ∅` at startup; each write in `_correct_divergence` adds its artifact.
- `_verify_consistency` runs conditional checks: `specs/` modified → `gybis-allium-gate = true`; `implementation` modified → `test_suite_passes = true`; `architecture.md` modified → `vsm_coherence = true`.
- Completion requires `all_conditional_checks_pass = true` in the state machine, the fixed-point loop, and the regression contract.

Deadlock avoidance: if a conditional check fails but zero term divergences remain, looping to IDENTIFYING_DIVERGENCES would stall — a broken artifact is not resolvable by term-divergence resolution. That case halts with a diagnostic naming the failed artifact and routes to the owning repair skill. [AMENDED — see conditional-verification-scope.]
