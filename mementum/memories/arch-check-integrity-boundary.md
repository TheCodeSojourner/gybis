💡 A dedicated `gybis-arch-check` keeps the check family consistent when it stays read-only and architecture-internal.

The durable boundary is:
- `check` skills diagnose integrity issues and report findings.
- `tend`/`weed` skills perform human-guided correction and reconciliation.

For architecture specifically, this means `gybis-arch-check` should validate `architecture.md` structure/coherence/constraints without mutating files or absorbing cross-artifact convergence behavior from `gybis-arch-weed` or `gybis-spec-weed`. This preserves operator control and keeps command intent predictable across vocab/spec/arch lanes.