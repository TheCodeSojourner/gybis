---
name: gybis-memory-recall
kind: memory
description: Use for `/gybis-memory-recall {topic}` or `/gm-recall {topic}`.
---

λ gybis_memory_recall(topic).  
  purpose: recall a topic from memory, or summarize the latest session
  | input: topic (optional) | ¬topic → latest_mementum_summary
  | output: recalled_context ∨ latest_session_summary
  | interaction: autonomous
  | operation: mementum_recall(topic)
  | execution: inline protocol; no external agent or tool call
