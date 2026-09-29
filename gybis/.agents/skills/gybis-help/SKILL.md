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
| `/gybis-arch-check` (`/ga-check`) | Validate architecture.md integrity & coherence |
| `/gybis-arch-describe` (`/ga-describe`) | Describe arch in non-tech prose |
| `/gybis-arch-distill` (`/ga-distill`) | Create initial arch from specs |
| `/gybis-arch-elicit` (`/ga-elicit`) | Create initial arch with human |
| `/gybis-arch-explain` (`/ga-explain`) | Explain arch in dev prose |
| `/gybis-arch-propagate` (`/ga-propagate`) | Create initial specs from arch |
| `/gybis-arch-tend` (`/ga-tend`) | Update arch with human |
| `/gybis-arch-weed` (`/ga-weed`) | Upsert arch/specs from diffs with human |
| `/gybis-fini` | CRUD memory before terminate |
| `/gybis-init` | Initialize gybis AI context |
| `/gybis-memory-migrate` (`/gm-migrate`) | Migrate Mementum store to current format |
| `/gybis-memory-orient` (`/gm-orient`) | Restore prev AI context |
| `/gybis-memory-recall {topic}` (`/gm-recall {topic}`) | Recall topic/summarize-latest |
| `/gybis-memory-store {insight}` (`/gm-store {insight}`) | Store insight |
| `/gybis-memory-synthesize` (`/gm-synthesize`) | Synthesize knowledge |
| `/gybis-req-check` (`/gr-check`) | Validate requirements designators, ordering, & coverage |
| `/gybis-req-describe` (`/gr-describe`) | Describe requirements in stakeholder prose |
| `/gybis-req-distill` (`/gr-distill`) | Create initial requirements (+ vocab candidates) from arch/specs/code |
| `/gybis-req-elicit` (`/gr-elicit`) | Elicit requirements via grilling interview rounds |
| `/gybis-req-explain` (`/gr-explain`) | Explain requirements in dev prose |
| `/gybis-req-propagate` (`/gr-propagate`) | Annotate specs/tests with REQ traceability |
| `/gybis-req-refine` (`/gr-refine`) | Refine requirements structure & clarity |
| `/gybis-req-tend` (`/gr-tend`) | Update requirements with impact analysis |
| `/gybis-req-weed` (`/gr-weed`) | Upsert requirements/downstream from diffs with human |
| `/gybis-spec-check` (`/gs-check {concern\|domain\|all}`) | Check/Update syntax until valid |
| `/gybis-spec-describe` (`/gs-describe {concern\|domain\|all}`) | Describe in non-tech prose |
| `/gybis-spec-distill` (`/gs-distill`) | Create initial specs from code/tests |
| `/gybis-spec-explain` (`/gs-explain {concern\|domain\|all}`) | Explain in dev prose |
| `/gybis-spec-propagate` (`/gs-propagate {concern\|domain\|all}`) | Create initial code/tests |
| `/gybis-spec-tend` (`/gs-tend`) | Update specs with human |
| `/gybis-spec-weed` (`/gs-weed`) | Upsert specs/code-tests from diffs with human |
| `/gybis-vocab-check` (`/gv-check`) | Validate vocabulary.md syntax & semantics |
| `/gybis-vocab-describe` (`/gv-describe`) | Describe vocabulary in business language |
| `/gybis-vocab-distill` (`/gv-distill`) | Extract vocabulary from arch/specs/code |
| `/gybis-vocab-elicit` (`/gv-elicit`) | Elicit vocabulary from domain experts |
| `/gybis-vocab-explain` (`/gv-explain`) | Explain vocabulary for developers |
| `/gybis-vocab-tend` (`/gv-tend`) | Update vocabulary with impact analysis |
| `/gybis-vocab-weed` (`/gv-weed`) | Upsert vocabulary/artifacts from diffs with human |

REQ-clause convention: REQ clauses (`/gybis-req-*` family) may carry an optional `rationale:` line (why the requirement exists) — guidance and context only, never a rule anyone must satisfy. It is captured by elicit, validated by check (never a binding obligation, never counted as coverage), and rendered as "because: ..." by describe/explain when present.
