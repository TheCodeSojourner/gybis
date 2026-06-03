---
name: gybis-memory-session-terminate
description: Use for `/gybis-memory-session-terminate` or `/gm-session-terminate`.
---

λ gybis_memory_session_terminate(). 
  p1:(read(mementum/state.md) → follow(related) → search(relevant) → read(needed))→id(task,questions,decisions,next)
  →p2:mementum_synthesize()
  →p3:upsert(state.md){last_session_id,current_timestamp,recover:next[1],task,questions,decisions,next}→"⏹→state.md"
  | path ∈ {mementum/state.md} | ¬∃mkdir ∧ ¬∃mkpath | write_only
  →p4:git_preserves_all→git_add(mementum/)→git_commit(message="session: {last_session_id} — {task[0]}")
