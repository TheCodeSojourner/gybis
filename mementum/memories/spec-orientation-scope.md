💡 spec orientation scope belongs to gybis-spec-propagate and gybis-spec-weed

FP/OOP orientation and language implications are architecture-level signals that should influence generation and divergence resolution, not generic spec validation or documentation.

Scope decision:
- Keep orientation-aware behavior in `/gybis-spec-propagate` and `/gybis-spec-weed`.
- Keep `read(architecture.md)` for those two skills as source input.
- Do not require orientation handling in `/gybis-spec-check`, `/gybis-spec-describe`, `/gybis-spec-distill`, `/gybis-spec-explain`, or `/gybis-spec-tend`.

Why:
- Prevents coupling low-level or read-only spec workflows to architecture preferences.
- Keeps deterministic/read-only skills focused on their core contract.
- Preserves clear ownership: architecture preferences drive propagation and reconciliation.