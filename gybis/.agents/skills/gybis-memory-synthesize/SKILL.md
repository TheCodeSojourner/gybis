---
name: gybis-memory-synthesize
kind: memory
description: Use for `/gybis-memory-synthesize` or `/gm-synthesize`.
disable-model-invocation: true
---

λ gybis_memory_synthesize().
  purpose: synthesize knowledge from accumulated memories
  | input: none
  | output: synthesized_knowledge
  | interaction: autonomous
  | operation: mementum_synthesize()
  | execution: inline protocol; no external agent or tool call

λ gybis-memory-synthesize_candidate(topic).
  evidence ≔ recall(topic)
  | synthesize_when(card(memories(topic)) ≥ 3 ∨ stale_memory(topic) ∨ crystallized_understanding(topic))
  | draft(type = Reference, title, description, status = open, related)
  | human_approval = true → create_or_update(mementum/knowledge/{topic}.md)
  | human_approval = false → preserve_draft ∧ report_pending
