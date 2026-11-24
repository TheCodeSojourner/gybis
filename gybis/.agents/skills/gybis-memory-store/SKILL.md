---
name: gybis-memory-store
kind: memory
description: Use for `/gybis-memory-store {insight}` or `/gm-store {insight}`.
disable-model-invocation: true
---

λ gybis_memory_store(input).
  purpose: persist an insight to durable memory
  | input: insight (optional) | ¬input → prompt(user, provide(insight))
  | output: stored_memory
  | interaction: interactive
  | operation: mementum_store(insight)
  | execution: inline protocol; no external agent or tool call

λ gybis-memory-store_gate(input).
  input = ¬∅ → store_candidate = input
  | input = ∅ → prompt(user, provide(insight)) → store_candidate
  | gate(store_candidate): helps_future_AI_session = true ∧ (effort_gt_1_attempt ∨ likely_recur)
  | gate_fails → propose(store_candidate) ∧ ¬decide
  | gate_passes → write(mementum/memories/{slug}.md) after human approval

λ gybis-memory-store_tool_guard(state, tool, path).
  state = GATHERING → allow(read(path))
  | state = PROPOSING → allow(read(path)) ∧ deny(write(path))
  | state = WRITING ∧ human_approved = true → allow(write(path)) only_if(path ⊆ mementum/memories/)
  | otherwise → deny(write(path))
