---
type: Decision
symbol: 🎯
title: All four weeders gate completion on validity of what they wrote
related: vocab-weed-validity-gate, convergence-and-deliver-uniformity, check-boundary-verifier-exception, reference-corpus-cleanup
---

All four weeders gate completion on validity of whatever they wrote, not just on agreement.

Pattern (vocab-weed-validity-gate, extended to req-weed):
- `modified_artifacts ≔ ∅` at startup; each write branch records its artifact.
- `_verify` runs conditional checks keyed to what was modified: `specs/` → `gybis-allium-gate = true`; `architecture.md` → `vsm_coherence = true`; `implementation`/`test_paths` → `test_suite_passes = true`.
- Completion requires `all_conditional_checks_pass = true` in the state machine and the regression contract.
- Deadlock avoidance: if a check fails but zero divergences remain, looping would stall (a broken artifact is not resolvable by divergence resolution). That case halts, names the failed artifact, and routes to the owning repair skill.

req-weed's gap: it could write `specs/` and `architecture.md` via `resolve_mode = spec|arch` while verifying only term divergences, designator integrity, and tests — so it could break Allium syntax or VSM coherence and still report COMPLETE.

Final scope: arch-weed verifies allium-gate + vsm_coherence unconditionally; spec-weed adds coverage + strict_coverage + tests; req-weed and vocab-weed verify conditionally on what they modified.

Incidental normalisation: req-weed computes `remaining ≔ card(remaining_divergences)` once.
