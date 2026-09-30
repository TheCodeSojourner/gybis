---
type: Decision
symbol: 🎯
title: distill-input-direction
related: requirements-layer-family, interaction-mode-taxonomy, gybis-mementum-separate
---

Brownfield distill inputs are strictly bottom-up: each layer is distilled from the layers below it, never from a layer above.

Input sets:
- `spec-distill`: implementation (including test files)
- `arch-distill`: specs + implementation
- `vocab-distill`: architecture + specs + implementation
- `req-distill`: vocabulary + architecture + specs + implementation

Direction is now vocabulary → requirements.

Superseded by this decision:
- the earlier clause "gybis-vocab-distill now lists requirements/ as a source" in `requirements-layer-family`
- the earlier brownfield order `spec-distill → arch-distill → gr-distill` in `requirements-layer`

Residual, deliberate:
- `req-distill` still emits term candidates to `/gybis-vocab-tend`, but only for terms ∉ vocabulary.md — a safety valve for terms discovered during transcription, not the primary direction.
- `req-distill` reading vocabulary.md is a read of a lower layer. Distillation inverts the constraint order, so this is not a reverse dependency in the constraint sense: the layer order `requirements > vocabulary > architecture > specs > tests > code` is unchanged.
