---
name: gybis-help
kind: help
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
| `/gybis-arch-check` (`/ga-check`) | Validate architecture.md integrity & coherence |
| `/gybis-arch-describe` (`/ga-describe`) | Describe arch in stakeholder prose |
| `/gybis-arch-distill` (`/ga-distill`) | Create initial arch from specs |
| `/gybis-arch-explain` (`/ga-explain`) | Explain arch in dev prose |
| `/gybis-arch-propagate` (`/ga-propagate`) | Create initial specs from arch |
| `/gybis-arch-tend` (`/ga-tend`) | Update arch with impact analysis |
| `/gybis-arch-weed` (`/ga-weed`) | Resolve divergence between arch and specs |
| `/gybis-fini` | Persist memory before terminate |
| `/gybis-init` | Initialize gybis AI context |
| `/gybis-memory-migrate` (`/gm-migrate`) | Migrate Mementum store to current format |
| `/gybis-memory-orient` (`/gm-orient`) | Restore prev AI context |
| `/gybis-memory-recall {topic}` (`/gm-recall {topic}`) | Recall topic/summarize-latest |
| `/gybis-memory-store {insight}` (`/gm-store {insight}`) | Store insight |
| `/gybis-memory-synthesize` (`/gm-synthesize`) | Synthesize knowledge |
| `/gybis-req-check` (`/gr-check`) | Validate individual requirements, ordering, & coverage |
| `/gybis-req-describe` (`/gr-describe`) | Describe requirements in stakeholder prose |
| `/gybis-req-distill` (`/gr-distill`) | Create initial requirements from vocab/arch/specs/code |
| `/gybis-req-elicit` (`/gr-elicit`) | Elicit requirements via grilling interview rounds |
| `/gybis-req-explain` (`/gr-explain`) | Explain requirements in dev prose |
| `/gybis-req-propagate` (`/gr-propagate`) | Annotate specs/tests with REQ traceability |
| `/gybis-req-refine` (`/gr-refine`) | Refine requirements structure & clarity |
| `/gybis-req-tend` (`/gr-tend`) | Update requirements with impact analysis |
| `/gybis-req-weed` (`/gr-weed`) | Resolve divergence between requirements and downstream |
| `/gybis-spec-check` (`/gs-check {concern\|domain\|all}`) | Validate and repair spec syntax |
| `/gybis-spec-describe` (`/gs-describe {concern\|domain\|all}`) | Describe specs in stakeholder prose |
| `/gybis-spec-distill` (`/gs-distill`) | Create initial specs from code/tests |
| `/gybis-spec-explain` (`/gs-explain {concern\|domain\|all}`) | Explain specs in dev prose |
| `/gybis-spec-propagate` (`/gs-propagate {concern\|domain\|all}`) | Create initial code/tests |
| `/gybis-spec-tend` (`/gs-tend`) | Update specs with impact analysis |
| `/gybis-spec-weed` (`/gs-weed`) | Resolve divergence between specs and code |
| `/gybis-vocab-check` (`/gv-check`) | Validate vocabulary.md syntax & semantics |
| `/gybis-vocab-describe` (`/gv-describe`) | Describe vocabulary in stakeholder prose |
| `/gybis-vocab-distill` (`/gv-distill`) | Extract vocabulary from arch/specs/code |
| `/gybis-vocab-explain` (`/gv-explain`) | Explain vocabulary in dev prose |
| `/gybis-vocab-propagate` (`/gv-propagate`) | Create initial architecture from req + vocab |
| `/gybis-vocab-tend` (`/gv-tend`) | Update vocabulary with impact analysis |
| `/gybis-vocab-weed` (`/gv-weed`) | Resolve divergence between vocab and downstream |
