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
  | delegate: mementum_store(insight)
