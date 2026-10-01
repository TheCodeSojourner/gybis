---
type: Insight
symbol: 💡
title: developer-owned-stage-readiness
---

Developer-owned stage readiness works best when skill ownership boundaries are explicit

If gybis is command-driven guidance (not always-on enforcement), then readiness for each layer (`vocabulary -> architecture -> specs -> code/tests`) should be owned by the developer, while each skill owns only its transformation scope.

Practical boundary that stayed coherent in this session:
- `gybis-vocab-*` owns vocabulary convergence and drift resolution.
- `gybis-spec-*` owns behavior/spec/code/arch consistency, not vocabulary policing.
- `gybis-arch-elicit` was removed (session-42); greenfield `architecture.md` is bootstrapped by `/gybis-vocab-propagate` and validated by `/gybis-arch-check`, which does not hard-halt on other artifacts. [AMENDED — see memories/propagate-seed-then-own.md: propagation bootstraps the layer, then tend/refine/weed own it; stage readiness remains developer-owned.]

This keeps prompts lean, avoids cross-skill concern leakage, and preserves developer control while still providing explicit convergence tools (`check`/`weed`) when the developer chooses to run them.