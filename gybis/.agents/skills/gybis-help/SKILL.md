---
name: gybis-help
description: Use for `/gybis-help`.
---

Print the following table directly in the AI response. Do not describe the table or its contents, just print the markdown for the table itself.

CRITICAL CONSTRAINTS:
1. The table must be an exact verbatim copy of the markdown below — do not modify, reword, reformat, or paraphrase any cell content in either column.
2. Do NOT add any preamble before the table.
3. Do NOT add any text, summary, follow-up, or questions after the closing code fence.
4. The response must contain nothing except the code-fenced markdown table and nothing else.

```markdown
| Skill Name | Description |
|---|---|
| `/gybis-arch-describe` (`/ga-describe`) | Describe architecture in product-manager-friendly prose. |
| `/gybis-arch-distill` (`/ga-distill`) | Create an initial architecture from specs. |
| `/gybis-arch-elicit` (`/ga-elicit`) | Create an initial architecture with guided interaction. |
| `/gybis-arch-explain` (`/ga-explain`) | Explain architecture in developer prose. |
| `/gybis-arch-tend` (`/ga-tend`) | Guide interactive update of architecture. |
| `/gybis-arch-weed` (`/ga-weed`) | Analyze architecture (source of truth) vs. specs divergence. |
| `/gybis-memory-orient` (`/gm-orient`) | Restore AI context from memory information. |
| `/gybis-memory-recall {topic}` (`/gm-recall {topic}`) | Recall memory information by optional topic, or write a brief summary. |
| `/gybis-memory-session-terminate` (`/gm-session-terminate`) | CRUD memory information prior to session termination. |
| `/gybis-memory-store {insight}` (`/gm-store {insight}`) | Store an optional insight as a memory, or prompt for one. |
| `/gybis-memory-synthesize` (`/gm-synthesize`) | Invoke memory-to-knowledge synthesis. |
| `/gybis-spec-check` (`/gs-check`) | Check/Update spec file/domain/all syntax until validated. |
| `/gybis-spec-describe` (`/gs-describe`) | Describe spec file/domain/all in product-manager-friendly prose. |
| `/gybis-spec-distill` (`/gs-distill`) | Create initial specs from tests/code. |
| `/gybis-spec-elicit` (`/gs-elicit`) | Create initial specs with guided interaction. |
| `/gybis-spec-explain` (`/gs-explain`) | Explain spec file/domain/all in developer prose. |
| `/gybis-spec-propagate` (`/gs-propagate`) | Generate/Update test files from spec(s). |
| `/gybis-spec-tend` (`/gs-tend`) | Guide interactive update of file/domain/all. |
| `/gybis-spec-weed` (`/gs-weed`) | Analyze specs (source of truth) vs. tests/code divergence. |
```
