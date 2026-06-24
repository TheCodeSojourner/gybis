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
| `/gybis-fini` | CRUD memory before terminate |
| `/gybis-init` | Initialize AI context |
| `/gybis-mementum-orient` (`/gm-orient`) | Restore prev AI context |
| `/gybis-mementum-recall {topic}` (`/gm-recall {topic}`) | Recall topic/summarize-latest |
| `/gybis-mementum-store {insight}` (`/gm-store {insight}`) | Store insight |
| `/gybis-mementum-synthesize` (`/gm-synthesize`) | Synthesize knowledge |

