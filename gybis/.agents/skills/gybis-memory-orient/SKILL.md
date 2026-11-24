---
name: gybis-memory-orient
kind: memory
description: Use for `/gybis-memory-orient` or `/gm-orient`.
disable-model-invocation: true
---

λ gybis_memory_orient().
  purpose: restore previous AI context from the mementum store
  | input: optional current_task_terms
  | output: restored_context ∧ orient_manifest
  | interaction: autonomous
  | operation: mementum_orient(current_task_terms)
  | execution: inline protocol; no external agent or tool call
  | always_load:
      mementum/state.md
      mementum/index.md
      frontmatter metadata for mementum/memories/ ∧ mementum/knowledge/
      current open ∧ active questions
  | relevance_load:
      recent memories since last_session_id
      knowledge pages related to current_task_terms
      memories linked through related frontmatter
      exact-term Git grep hits
      Git history only when temporal recency is relevant
  | recall:
      exact_terms → follow(related) → widen_incrementally_if_empty_or_thin
      stop_when_sufficient
  | fixed_memory_minimum: ¬∃
  | manifest:
      state_read ∧ index_read ∧ frontmatter_metadata_scanned
      ∧ focused_memories_read ∧ focused_knowledge_read
      ∧ searches_run ∧ unresolved_questions ∧ relevant_content_not_loaded

λ gybis-memory-orient_state_machine(state, action).
  state ∈ {INIT, ALWAYS_CONTEXT, RELEVANCE_SEARCH, RELATED_TRAVERSAL, SUFFICIENT, COMPLETE}
  | transition(INIT → ALWAYS_CONTEXT) only_if(state_read ∧ index_read ∧ metadata_scanned)
  | transition(ALWAYS_CONTEXT → RELEVANCE_SEARCH) only_if(open_active_questions_loaded)
  | transition(RELEVANCE_SEARCH → RELATED_TRAVERSAL) only_if(exact_hits ∃ ∨ related_edges ∃)
  | transition(RELEVANCE_SEARCH → SUFFICIENT) only_if(exact_hits = ∅ ∧ related_edges = ∅)
  | transition(RELATED_TRAVERSAL → SUFFICIENT) only_if(neighborhood_sufficient = true)
  | transition(SUFFICIENT → COMPLETE) only_if(orient_manifest_complete = true)

λ gybis-memory-orient_recall(current_task_terms).
  exact(current_task_terms) → follow(related) → read_temporal_history_if_relevant
  | empty_or_thin(exact) → widen(adjacent_topics) incrementally
  | stop_when_sufficient = true
  | never_load_full_bundle_to_satisfy_fixed_limit
