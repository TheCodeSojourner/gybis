💡 Human-owned stage readiness works best when skill ownership boundaries are explicit

If gybis is command-driven guidance (not always-on enforcement), then readiness for each layer (`vocabulary -> architecture -> specs -> code/tests`) should be owned by the human operator, while each skill owns only its transformation scope.

Practical boundary that stayed coherent in this session:
- `gybis-vocab-*` owns vocabulary convergence and drift resolution.
- `gybis-spec-*` owns behavior/spec/code/arch consistency, not vocabulary policing.
- `gybis-arch-elicit` may use vocabulary context when present, but should not hard-halt when missing.

This keeps prompts lean, avoids cross-skill concern leakage, and preserves operator control while still providing explicit convergence tools (`check`/`weed`) when the human chooses to run them.