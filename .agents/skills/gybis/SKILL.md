---
name: gybis
description: Use for `/gybis`.
---

Print the following table directly in the AI response. Do not describe the table or its contents, just print the markdown for the table itself.

```markdown
| Skill Name | Description |
|---|---|
| `/gybis-mementum-orient` (`/gm-orient`) | Restore AI context from latest mementum information. Note: This happens automatically when a new session starts, but this command can be used to manually trigger context restoration. |
| `/gybis-mementum-recall {topic}` (`/gm-recall {topic}`) | Recall mementum information by optional topic. If no topic is provided, it writes a brief summary of the latest mementum information. |
| `gybis-mementum-session-terminate` (`/gm-session-terminate`) | Closes out a session, prior to termination, in a lossless, handoff-ready way so work can resume cleanly in the next session. |
| `/gybis-mementum-store {insight}` (`/gm-store {insight}`) | Store an insight as a mementum memory. If insight is not provided, it prompts the user. |
| `/gybis-mementum-synthesize` (`/gm-synthesize`) | Invokes the mementum memory synthesis workflow to synthesize memories into knowledge if needed. |
```