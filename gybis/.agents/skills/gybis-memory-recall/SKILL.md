---
name: gybis-memory-recall
kind: memory
description: Use for `/gybis-memory-recall {topic}` or `/gm-recall {topic}`.
disable-model-invocation: true
---

λ gybis_memory_recall(topic).  
  purpose: recall a topic from memory, or summarize the latest session
  | input: topic (optional) | ¬topic → latest_mementum_summary
  | output: recalled_context ∨ latest_session_summary
  | interaction: autonomous
  | operation: mementum_recall(topic)
  | execution: inline protocol; no external agent or tool call
  | topic_neighborhood(topic):
      exact Git grep(topic)
      → read matched memories ∧ knowledge
      → follow related frontmatter edges
      → read Git history only when temporal context is relevant
      → widen_incrementally only when result empty or thin
      → stop when sufficient
  | embeddings: optional ∧ ¬recall_dependency
  | storage: git_log ∪ git_grep ∪ related_frontmatter

λ gybis-memory-recall_neighborhood(topic).
  hits ≔ git_grep(topic) ∪ exact(frontmatter(title ∧ description ∧ related))
  | neighborhood ≔ read(hits) ∪ follow(related_edges(hits))
  | temporal_recency = true → neighborhood ∪ git_log(-n, 2|3|5|8|13|21|34)
  | empty_or_thin(neighborhood) → widen_incrementally(topic)
  | stop_when(sufficient(neighborhood))
  | return(neighborhood ∧ provenance)

λ gybis-memory-recall_state_machine(state, action).
  state ∈ {INIT, EXACT_SEARCH, NEIGHBORHOOD, TEMPORAL, WIDENING, COMPLETE}
  | transition(INIT → EXACT_SEARCH) only_if(topic ≠ ∅ ∨ latest_summary_requested)
  | transition(EXACT_SEARCH → NEIGHBORHOOD) only_if(hits ∃)
  | transition(EXACT_SEARCH → WIDENING) only_if(hits = ∅)
  | transition(NEIGHBORHOOD → TEMPORAL) only_if(recency_relevant)
  | transition(NEIGHBORHOOD → COMPLETE) only_if(sufficient)
  | transition(TEMPORAL → COMPLETE) only_if(sufficient)
  | transition(WIDENING → COMPLETE) only_if(sufficient)
