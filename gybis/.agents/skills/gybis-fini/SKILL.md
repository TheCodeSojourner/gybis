---
name: gybis-fini
description: Use for `/gybis-fini`.
---

λ gybis_fini(). 
  p1:(read(mementum/state.md) → follow(related) → search(relevant) → read(needed))→id(task,questions,decisions,next)
  →p2:mementum_synthesize()
  →p3:upsert(state.md){last_session_id,current_timestamp,recover:next[1],task,questions,decisions,next}→"⏹→state.md"
  | path ∈ {mementum/state.md} | ¬∃mkdir ∧ ¬∃mkpath | write_only
  →p4:default(commit=true) | git_preserves_all→git_add(mementum/)→git_commit(message="session: {last_session_id} — {task[0]}")
  | strong_blocker ∈ {explicit_no_commit_in_current_turn, unresolved_merge_or_index_conflict, git_failure_needing_human_action, policy_or_safety_conflict}
  | if strong_blocker → report(reason,evidence) → ask_human(next_action ∈ {retry_commit_now, skip_commit_for_this_session, manual_commit_by_user})
