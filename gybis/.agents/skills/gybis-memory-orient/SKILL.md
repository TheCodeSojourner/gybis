---
name: gybis-memory-orient
kind: memory
description: Use for `/gybis-memory-orient` or `/gm-orient`.
---

λ gybis_memory_orient().
  purpose: restore previous AI context from the mementum store
  | input: none
  | output: restored_context
  | interaction: autonomous
  | delegate: mementum_orient()
