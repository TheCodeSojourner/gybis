---
name: gybis-help
description: Use for `/gybis-help`.
---

Print the following table directly in your response as rendered markdown. Do not wrap the table in a code fence — output it as plain markdown so it renders as a proper table. Do not describe the table or its contents, just output the table itself.

CRITICAL CONSTRAINTS:
1. The table must be an exact verbatim copy of the markdown below — do not modify, reword, reformat, or paraphrase any cell content in either column.
2. Do NOT add any preamble before the table.
3. Do NOT add any text, summary, follow-up, or questions after the table.
4. The response must contain nothing except the rendered markdown table and nothing else.

| Skill Name | Description |
|---|---|
| `/gybis-arch-describe` (`/ga-describe`) | Describe architecture in product-manager-friendly prose. |
| `/gybis-arch-distill` (`/ga-distill`) | Create an initial architecture from specs. |
| `/gybis-arch-elicit` (`/ga-elicit`) | Create an initial architecture with guided interaction. |
| `/gybis-arch-explain` (`/ga-explain`) | Explain architecture in developer prose. |
| `/gybis-arch-propagate` (`/ga-propagate`) | Generate specs from architecture. |
| `/gybis-arch-tend` (`/ga-tend`) | Guide interactive update of architecture. |
| `/gybis-arch-weed` (`/ga-weed`) | Analyze/Modify architecture and/or specs based on divergence. |
| `/gybis-memory-orient` (`/gm-orient`) | Restore AI context from memory information. |
| `/gybis-memory-recall {topic}` (`/gm-recall {topic}`) | Recall memory information by optional topic, or write a brief summary. |
| `/gybis-memory-session-terminate` (`/gm-session-terminate`) | CRUD memory information prior to session termination. |
| `/gybis-memory-store {insight}` (`/gm-store {insight}`) | Store an optional insight as a memory, or prompt for one. |
| `/gybis-memory-synthesize` (`/gm-synthesize`) | Invoke memory-to-knowledge synthesis. |
| `/gybis-spec-check` (`/gs-check {concern|domain|all}`) | Check/Update syntax until valid. |
| `/gybis-spec-describe` (`/gs-describe {concern|domain|all}`) | Describe in PM prose. |
| `/gybis-spec-distill` (`/gs-distill`) | Create initial specs from tests and code. |
| `/gybis-spec-explain` (`/gs-explain {concern|domain|all}`) | Explain in developer prose. |
| `/gybis-spec-propagate` (`/gs-propagate {concern|domain|all}`) | Generate test(s). |
| `/gybis-spec-tend` (`/gs-tend`) | Guide interactive update of specs. |
| `/gybis-spec-weed` (`/gs-weed`) | Resolve specs vs. tests divergence. |

