---
type: Insight
symbol: 💡
title: allium-353-adapter-compatibility
---

Start the next session with a focused Allium 3.5.3 adapter compatibility pass.

The direct upstream pins are already current (`allium` `527cd52`; `allium-tools`
`08d3139`), but a real CLI smoke test showed adapter drift in the bundled
internal skills:

- `allium check` and `allium analyse` emit top-level `diagnostics`, `findings`,
  `command`, and `spec_file`; exit code 1 can represent valid JSON diagnostics
  or findings.
- `allium plan`, `parse`, and `model` emit diagnostics alongside their primary
  data; nonzero exit with error diagnostics must be treated as a structured
  failure rather than malformed output.
- Update `internal/allium-check`, `internal/allium-analyse`,
  `internal/allium-plan`, and `internal/allium-normalize` to consume the actual
  v3.5.3 JSON shapes. Do not change upstream pins solely because of this work.

Keep this pass separate from the completed Nucleus Lambda/VSM update.