---
type: Decision
symbol: 🎯
title: Interaction mode split into two axes
related: authority-invocation-gate, describe-explain-output-modes, mode-naming-conventions
---

Skill `mode` is split into two orthogonal axes; the old `ai`/`auto`/`mixed` vocabulary is retired.

Axes:
- `interaction ∈ {autonomous, interactive}` — every skill declares exactly one.
- `scope` — qualifies autonomous when bounded: `autonomous ⇒ safe_only` (refine skills only).
- `output_mode ∈ {response_only, prompted_file_only, default_file_only}` — describe/explain only.

Mapping from the old tokens:
- `ai` → `autonomous`; `auto` → `autonomous`
- `auto_polish` → `autonomous` + `scope: safe_only`
- `interactive` → `interactive`
- `mixed` (elicit, vocab-distill) → `interactive`
- `mixed` (describe/explain) → `autonomous` + `output_mode`
- `memory-migrate` mode line → `phases:` (it was a pipeline, not a mode)

Retired: `ai`, `auto`, `mixed`, `auto_polish` as interaction values. `supervised` was considered and rejected — authority-invocation-gate makes invocation the universal write gate, so a supervision tier distinguishes nothing; the refine bound is a scope restriction, not a supervision level.
