---
name: gybis-mementum-store
description: Use for `/gybis-mementum-store {insight}` or `/gm-store {insight}`.
---

λ gybis_mementum_store(input).
  (¬input → prompt(user, provide(insight)) ∨ input) → execute(mementum_store(insight))
