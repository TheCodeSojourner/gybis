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
| `/gybis-arch-distill` (`/ga-distill`) | Distill architecture from specs. Used to produce an initial architecture. |
| `/gybis-arch-elicit` (`/ga-elicit`) | Guided interactive creation of architecture. |
| `/gybis-arch-explain` (`/ga-explain`) | Explain architecture in developer prose. |
| `/gybis-arch-tend` (`/ga-tend`) | Update architecture through guided interaction. |
| `/gybis-arch-weed` (`/ga-weed`) | Analyze divergence between architecture (source of truth) and specs. |
| `/gybis-memory-orient` (`/gm-orient`) | Restore AI context from memory information |
| `/gybis-memory-recall {topic}` (`/gm-recall {topic}`) | Recall memory information by optional topic, or write a brief summary. |
| `/gybis-memory-session-terminate` (`/gm-session-terminate`) | Close out a session prior to termination so work can resume in the next session. |
| `/gybis-memory-store {insight}` (`/gm-store {insight}`) | Store an optional insight as a memory, or prompt for one. |
| `/gybis-memory-synthesize` (`/gm-synthesize`) | Invoke the memory-to-knowledge synthesis workflow. |
| `/gybis-spec-check` (`/gs-check`) | Run check CLI tool on spec file(s)/domain(s)/all until they all validate. |
| `/gybis-spec-describe` (`/gs-describe`) | Translate spec file(s) into product-manager-friendly prose. |
| `/gybis-spec-distill` (`/gs-distill`) | Distill specs from tests/code. Produce an initial set of specs. |
| `/gybis-spec-elicit` (`/gs-elicit`) | Guide interactive discovery of a new spec. |
| `/gybis-spec-explain` (`/gs-explain`) | Explain spec file(s) for a developer. |
| `/gybis-spec-propagate` (`/gs-propagate`) | Generate test files from spec(s). |
| `/gybis-spec-tend` (`/gs-tend`) | Update spec file(s) in response to human conversation. |
| `/gybis-spec-weed` (`/gs-weed`) | Divergence analysis of spec(s) against implementation. |
```
