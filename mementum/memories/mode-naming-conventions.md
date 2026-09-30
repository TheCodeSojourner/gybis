---
type: Decision
symbol: 🎯
title: Mode-axis naming conventions and retained names
related: interaction-mode-taxonomy
---

Conventions for the two skill-mode axes, and the names deliberately left alone.

Naming rule: an identifier that would carry two different meanings across the skill family must be renamed; symmetry alone is not a reason to rename.

- The purpose block declares `| interaction:` (and `| output_mode:` where applicable); the lambda and gate remain the normative enforcement.
- Output axis: `_output_mode` / `_output_mode_gate(state, output_mode)`; field `output_modes`. Applied to all describe/explain skills.
- Interaction axis: the lambda stays `_mode` / `_mode_gate(state, mode)`; field `interaction_modes` (renamed from the vague `valid_modes`). `_mode` is kept deliberately — once the output axis no longer uses `_mode` it is unambiguous, so a 28-file rename adds churn without removing ambiguity.
- `modes:` in req describe/explain renamed to `output_modes:`.

Known remaining names (deliberately not changed):
- File-local variables `selected_mode`, `mode_selected`, `mode_selected_explicit`, `output_mode_choice`.
- State names `MODE_SELECTED`, `MODE_SELECTION` (per-file, always adjacent to the axis-specific lambda).
