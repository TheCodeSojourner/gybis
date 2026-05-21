---
name: gybis
description: Use for `/gybis`.
---

Print the following table directly in the AI response. Do not describe the table or its contents, just print the markdown for the table itself.

```markdown
| Skill Name | Description |
|---|---|
| `/gybis-arch-describe` (`/ga-describe`) | Translates `architecture.md` into a product-manager-friendly prose. |
| `/gybis-arch-distill` (`/ga-distill`) | Distills specs from `specs/` to architecture organized by VSM layer (S5→S1) in `architecture.md`. Produces an initial `architecture.md` from specs. |
| `/gybis-arch-elicit` (`/ga-elicit`) | Guides interactive discovery of system architecture via top-down VSM questioning (S5→S1), producing an initial `architecture.md` from a system vision. |
| `/gybis-arch-explain` (`/ga-explain`) | Explains an existing `architecture.md` for a developer, breaking down each VSM layer and cross-layer concerns. |
| `/gybis-arch-tend` (`/ga-tend`) | Updates `architecture.md` when requirements change. Human-driven revisions are mapped to correct VSM layers. |
| `/gybis-arch-weed` (`/ga-weed`) | Divergence analysis between `architecture.md` and `specs/`. Classifies each architectural element as Aligned, Partial, Contradicted, Missing, or Unspecified and presents findings grouped by VSM layer. |
| `/gybis-memory-orient` (`/gm-orient`) | Restore AI context from latest memory information. Note: This happens automatically when a new session starts, but this command can be used to manually trigger context restoration. |
| `/gybis-memory-recall {topic}` (`/gm-recall {topic}`) | Recall memory information by optional topic. If no topic is provided, it writes a brief summary of the latest memory information. |
| `/gybis-memory-session-terminate` (`/gm-session-terminate`) | Closes out a session, prior to termination, in a lossless, handoff-ready way so work can resume cleanly in the next session. |
| `/gybis-memory-store {insight}` (`/gm-store {insight}`) | Store an insight as a memory memory. If insight is not provided, it prompts the user. |
| `/gybis-memory-synthesize` (`/gm-synthesize`) | Invokes the memory memory synthesis workflow to synthesize memories into knowledge if needed. |
| `/gybis-spec-check` (`/gs-check`) | Runs `allium` CLI tool on spec file(s)/domain(s)/all in a loop until they all validate. |
| `/gybis-spec-describe` (`/gs-describe`) | Translates spec file(s) into product-manager-friendly prose. |
| `/gybis-spec-distill` (`/gs-distill`) | Distills specs from tests/code.  Produces an initial set of specs. |
| `/gybis-spec-elicit` (`/gs-elicit`) | Guides interactive discovery of a new spec. |
| `/gybis-spec-explain` (`/gs-explain`) | Explains spec file(s) for a developer. |
| `/gybis-spec-propagate` (`/gs-propagate`) | Generates test files from spec(s). |
| `/gybis-spec-tend` (`/gs-tend`) | Updates spec file(s) in response to human conversation. |
| `/gybis-spec-weed` (`/gs-weed`) | Divergence analysis of spec(s) against implementation. |
```
