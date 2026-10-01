---
type: Insight
symbol: 💡
title: Spec skills share one scope model
related: spec-weed-vocab-divergence-wiring, describe-explain-output-modes
---

All `gybis-spec-*` skills that operate on a subset of specifications use the same scope model. Reuse it; do not invent another.

Shape (defined as `<skill>_scope` + `<skill>_resolution`):

- `type ∈ {domain_concern, domain, all_specs} | default: all_specs`
- `domain ∧ concern → <root>/specs/{domain}/{concern}.allium`
- `domain ∧ ¬concern → <root>/specs/{domain}/`
- `¬domain ∧ ¬concern → <root>/specs/` (`"all"` / `"all specs"` resolve the same way)
- `scope_files ≔ recursive_files(scope_root, extension = .allium)`

Two rules that keep it coherent:

1. **The gate stays on existence, not scope.** `gate: specs/**/*.allium ∃ ∧ ¬∅` — a gate is a precondition evaluated before scope resolution, so referencing `scope_files` there is formally wrong. Scope governs the *work* (which files are read, checked, written, verified), never the precondition.
2. **Narrow every write guard to the scope.** `allow(write(path)) only_if(path ∈ scope_files)` — a scoped run must not mutate out-of-scope files.

Session-46 added this to `spec-check` and `spec-propagate`, which documented `{concern|domain|all}` in help tables but had no resolution lambda at all. `spec-describe`/`spec-explain` already had it.

Doc consequence: when the whole baseline is meant, omit the argument — the unscoped form *is* `all_specs`.
