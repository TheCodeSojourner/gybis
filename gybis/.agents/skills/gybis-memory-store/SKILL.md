---
name: gybis-memory-store
kind: memory
description: Use for `/gybis-memory-store {insight}` or `/gm-store {insight}`.
---

λ gybis_memory_store(input).
  purpose: persist an insight to durable memory
  | input: insight (optional) | ¬input → prompt(user, provide(insight))
  | output: stored_memory
  | interaction: interactive
  | operation: mementum_store(insight)
  | execution: inline protocol; no external agent or tool call
