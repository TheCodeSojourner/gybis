---
type: Insight
symbol: 💡
title: arch-check-integrity-boundary
---

A dedicated `gybis-arch-check` keeps the check family consistent when it stays read-only and architecture-internal.

Check-boundary rule: `check` diagnoses; `refine`/`tend`/`weed` act. Exception: `spec-check` also repairs, permitted because its corrections are verified by the external allium CLI (an oracle), which the other three check skills lack.

The durable boundary is:
- `check` skills diagnose integrity issues and report findings.
- `tend`/`weed` skills perform human-guided correction and reconciliation.

For architecture specifically, this means `gybis-arch-check` should validate `architecture.md` structure/coherence/constraints without mutating files or absorbing cross-artifact convergence behavior from `gybis-arch-weed` or `gybis-spec-weed`. This preserves operator control and keeps command intent predictable across vocab/spec/arch lanes.