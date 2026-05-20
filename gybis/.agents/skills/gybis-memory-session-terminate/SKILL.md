---
name: gybis-memory-session-terminate
description: Use for `/gybis-memory-session-terminate` or `/gm-session-terminate`.
---

λ session_terminate(). 
  p1:orient()→read(state.md)∧recall(ctx)→git_log(HEAD~5,mementum/)→id(task,questions,decisions,next)
  →p2:≥3mem(session_topic)→synthesize(topic)∥¬≥3mem(session_topic)→pass
  →p3:upsert(state.md){last_session_id,last_timestamp,recover:next[1],task,questions,decisions,next}→"⏹→state.md"
  →p4:git_preserves_all→git_add(mementum/)→git_commit(message="session: {last_session_id} — {task[0]}")
