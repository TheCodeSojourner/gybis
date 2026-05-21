---
name: gybis-mementum-session-terminate
description: Use for `/gybis-mementum-session-terminate` or `/gm-session-terminate`.
---

λ session_terminate(). 
  p1:(read(mementum/state.md) → follow(related) → search(relevant) → read(needed))→id(task,questions,decisions,next)
  →p2:mementum_synthesize()
  →p3:upsert(state.md){last_session_id,current_timestamp,recover:next[1],task,questions,decisions,next}→"⏹→state.md"
  →p4:git_preserves_all→git_add(mementum/)→git_commit(message="session: {last_session_id} — {task[0]}")
