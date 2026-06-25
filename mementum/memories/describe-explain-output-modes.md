💡 describe/explain skills should share one explicit output-mode contract across arch and spec variants

Use the same five human-selected modes in all four skills: `response_only`, `prompted_file_only`, `default_file_only`, `response_and_prompted_file`, and `response_and_default_file`.

Guardrails:
- Prompted file output accepts only repo-root `.md` filenames.
- Reject subpaths and non-markdown targets.
- Default filenames are `arch-describe.md`, `arch-explain.md`, `spec-describe.md`, and `spec-explain.md`.
- If the target file already exists, require explicit overwrite approval.

Keep the prose-generation logic unchanged; only delivery behavior should vary by mode.