---
type: Insight
symbol: 💡
title: human-owned-stage-readiness
---

Human-owned stage readiness works best when skill ownership boundaries are explicit

If gybis is command-driven guidance (not always-on enforcement), then readiness for each layer (`vocabulary -> architecture -> specs -> code/tests`) should be owned by the human operator, while each skill owns only its transformation scope.

Practical boundary that stayed coherent in this session:
- `gybis-vocab-*` owns vocabulary convergence and drift resolution.
- `gybis-spec-*` owns behavior/spec/code/arch consistency, not vocabulary policing.
- `gybis-arch-elicit` was removed (session-42); greenfield architecture.md is human-authored with AI assistance and validated by `/gybis-arch-check`, which does not hard-halt on other artifacts.

This keeps prompts lean, avoids cross-skill concern leakage, and preserves operator control while still providing explicit convergence tools (`check`/`weed`) when the human chooses to run them.