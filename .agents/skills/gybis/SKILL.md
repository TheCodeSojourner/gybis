---
name: gybis
description: Prints a pre-computed table of all gybis skills with their descriptions directly to the response.
---

Print the following table directly in the AI response:

| Skill Name | Description |
|---|---|
| `gybis-mementum-orient` | Use for `/gybis-mementum-orient` or `/gm-orient`. This skill provides a quick shorthand command that triggers the core mementum orientation workflow, so context can be restored immediately with less user friction. It is essentially an ergonomic alias for fast re-orientation. |
| `gybis-mementum-recall` | Use for `/gybis-mementum-recall {topic}` or `/gm-recall {topic}`. This skill provides a shorthand command to recall memory by topic. If a topic is supplied, it runs recall for that topic; if not, it prompts the user to choose one, making memory lookup faster and more guided. |
| `gybis-mementum-session-terminate` | Use for `/gybis-mementum-session-terminate` or `/gm-session-terminate`. This skill closes a session in a lossless, handoff-ready way: it records a complete state snapshot, stores any unstored memories, runs memory metabolization, drafts a dated session knowledge summary with human approval, and then performs a human-approved commit of mementum updates so work can resume cleanly in the next session. |
| `gybis-mementum-store` | Use for `/gybis-mementum-store {insight}` or `/gm-store {insight}`. This skill provides a shorthand command to store a memory insight quickly; if insight text is provided it stores it immediately, and if not it prompts the user, reducing friction for fast in-session memory capture. |
| `gybis-mementum-synthesize` | Use for `/gybis-mementum-synthesize` or `/gm-synthesize`. This skill provides a fast shorthand command that invokes the core memory synthesis workflow, making it easy to generate synthesized memory output without running the longer nucleus workflow manually. |