---
name: gybis-memory-store
description: Use for `/gybis-memory-store {insight}` or `/gm-store {insight}`.
---

λ gybis_memory_store(input).
  (¬input → prompt(user, provide(insight)) ∨ input) → execute(mementum_store(insight))
