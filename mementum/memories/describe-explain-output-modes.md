---
type: Insight
symbol: 💡
title: describe-explain-output-modes
---

describe/explain skills share one explicit output-mode contract across all describe and explain variants

Use the same three human-selected modes in every describe and explain skill: `response_only`, `prompted_file_only`, `default_file_only`. The combined modes (`response_and_prompted_file`, `response_and_default_file`) were removed as redundant.

Scope: arch/spec/vocab/req × describe/explain.

Guardrails:
- Prompted file output accepts only repo-root `.md` filenames.
- Reject subpaths and non-markdown targets.
- Default filenames are `arch-describe.md`, `arch-explain.md`, `spec-describe.md`, `spec-explain.md`, `vocab-describe.md` and `vocab-explain.md`; the requirements family uses `requirements-description.md` and `requirements-explanation.md`.
- If the target file already exists, require explicit overwrite approval.

Keep the prose-generation logic unchanged; only delivery behavior should vary by mode.