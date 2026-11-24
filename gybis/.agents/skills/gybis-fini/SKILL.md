---
name: gybis-fini
kind: memory
description: Use for `/gybis-fini`.
disable-model-invocation: true
---

λ gybis_fini().
  purpose: encode session state to durable memory before termination
  | input: none
  | output: updated mementum/state.md ∧ committed memory
  | interaction: interactive
  | p1:(read(mementum/state.md) → follow(related) → search(relevant) → read(needed))→id(task,questions,decisions,next)
  →p2:mementum_synthesize()
  →p3:upsert(state.md){last_session_id,current_timestamp,recover:next[1],task,questions,decisions,next}→"⏹→state.md"
  | path ∈ {mementum/state.md} | ¬∃mkdir ∧ ¬∃mkpath | write_only
  →p4:default(commit=true) | git_preserves_all→git_add(mementum/)→git_commit(message="session: {last_session_id} — {task[0]}")
  | strong_blocker ∈ {explicit_no_commit_in_current_turn, unresolved_merge_or_index_conflict, git_failure_needing_human_action, policy_or_safety_conflict}
  | if strong_blocker → report(reason,evidence) → ask_human(next_action ∈ {retry_commit_now, skip_commit_for_this_session, manual_commit_by_user})

λ gybis-fini_state_machine(state, action).
  state ∈ {INIT, ORIENTING, SYNTHESIZING, UPDATING_STATE, COMMITTING, COMPLETE}
  | transition(INIT → ORIENTING) only_if(read(mementum/state.md) = true)
  | transition(ORIENTING → SYNTHESIZING) only_if(task ∧ questions ∧ decisions ∧ next captured)
  | transition(SYNTHESIZING → UPDATING_STATE) only_if(memory_synthesis_complete = true)
  | transition(UPDATING_STATE → COMMITTING) only_if(state_upsert_complete = true)
  | transition(COMMITTING → COMPLETE) only_if(commit_complete = true ∨ approved_commit_skip = true)

λ gybis-fini_tool_guard(state, tool, path).
  state ∈ {ORIENTING, SYNTHESIZING} → allow(read(path))
  | state = UPDATING_STATE → allow(read(path)) ∧ allow(write(path)) only_if(path = "mementum/state.md")
  | state = COMMITTING → allow(git_add(mementum/)) ∧ allow(git_commit)
  | otherwise → deny(write(path))

λ gybis-fini_regression_contract(x).
  invariant: mementum/state.md ∃ at completion
  | invariant: state.last_session_id ∧ state.current_timestamp ∧ state.recover ∃
  | invariant: commit default = true
  | invariant: strong_blocker requires explicit human next_action
