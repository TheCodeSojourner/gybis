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
| `/gybis-mementum-orient` (`/gm-orient`) | Restore AI context from mementum information. |
| `/gybis-mementum-recall {topic}` (`/gm-recall {topic}`) | Recall mementum information by optional topic, or write a brief summary. |
| `/gybis-mementum-session-terminate` (`/gm-session-terminate`) | CRUD mementum information prior to session termination. |
| `/gybis-mementum-store {insight}` (`/gm-store {insight}`) | Store an optional insight as a mementum, or prompt for one. |
| `/gybis-mementum-synthesize` (`/gm-synthesize`) | Invoke mementum-to-knowledge synthesis. |

