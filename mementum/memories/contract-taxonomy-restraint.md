---
type: Decision
symbol: 🎯
title: Contract taxonomy restraint — declare only what is enforced
related: skill-contract-checker-rationale
---

Taxonomy restraint — the checker's principle (declare only what is enforced) applied to skill fields.

- `kind` is the only dimension genuinely invisible today, so it earns a frontmatter field.
- `mutates`, `verifier`, `read/write authority` are derivable from `tool_guard` / `_verify` / `_boundaries`. Declaring them would create a second source of truth that can disagree with the first — the `allium_gate` pattern. The checker derives them; no fields.
- `layer` is deliberately NOT a field: the skill name already encodes it.

Field trap avoided: `kind` is partially name-derivable (`gybis-spec-*` → domain). It is still worth a field because the checker validates declared kind against the name-derived class — two independent sources that must agree. This is the `interaction:` + `mode_gate` pattern (declared + enforced), not the `mode: mixed` pattern (declared only).

Session-close state: 41 user-facing skills — 33 domain, 7 memory, 1 help.
