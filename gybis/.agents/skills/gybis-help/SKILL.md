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
| `/gybis-arch-describe` (`/ga-describe`) | Describe arch in non-tech prose |
| `/gybis-arch-distill` (`/ga-distill`) | Create initial arch from specs |
| `/gybis-arch-elicit` (`/ga-elicit`) | Create initial arch with human |
| `/gybis-arch-explain` (`/ga-explain`) | Explain arch in dev prose |
| `/gybis-arch-propagate` (`/ga-propagate`) | Create initial specs from arch |
| `/gybis-arch-tend` (`/ga-tend`) | Update arch with human |
| `/gybis-arch-weed` (`/ga-weed`) | Upsert arch/specs from diffs with human |
| `/gybis-fini` | CRUD memory before terminate |
| `/gybis-init` | Initialize gybis AI context |
| `/gybis-memory-orient` (`/gm-orient`) | Restore prev AI context |
| `/gybis-memory-recall {topic}` (`/gm-recall {topic}`) | Recall topic/summarize-latest |
| `/gybis-memory-store {insight}` (`/gm-store {insight}`) | Store insight |
| `/gybis-memory-synthesize` (`/gm-synthesize`) | Synthesize knowledge |
| `/gybis-spec-check` (`/gs-check {concern\|domain\|all}`) | Check/Update syntax until valid |
| `/gybis-spec-describe` (`/gs-describe {concern\|domain\|all}`) | Describe in non-tech prose |
| `/gybis-spec-distill` (`/gs-distill`) | Create initial specs from code/tests |
| `/gybis-spec-explain` (`/gs-explain {concern\|domain\|all}`) | Explain in dev prose |
| `/gybis-spec-propagate` (`/gs-propagate {concern\|domain\|all}`) | Create initial code/tests |
| `/gybis-spec-tend` (`/gs-tend`) | Update specs with human |
| `/gybis-spec-weed` (`/gs-weed`) | Upsert specs/code-tests from diffs with human |

