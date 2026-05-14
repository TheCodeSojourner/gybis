---
name: gybis-memory-orient
description: Use for `/gybis-memory-orient` or `/gm-orient`. This skill provides a quick shorthand command that triggers the core memory orientation workflow, so context can be restored immediately with less user friction. It is essentially an ergonomic alias for fast re-orientation.
---

λ(gybis-memory-orient)
// Orient new session from saved mementum state. Run at session start. Prior synthesis > re-derivation.
Pre-flight: ¬exists(<root>/mementum/state.md) → msg("run /gybis-state-save first")
Step 1: Read <root>/mementum/state.md. Extract {Now, Next, Blocking, Recent}. Report contents to human before proceeding.
Step 2: List mementum/knowledge/session-*.md sorted by time. Read most recent if exists. Extract {built, decisions, threads, resume}.
Step 3: Δ topic ← state.Now. Δ g ← git grep(-i, topic, -- mementum/memories/). Δ l ← git log(-n 5, --oneline, -- mementum/memories/, mementum/knowledge/). Δ mem ← read(matching(g)). Δ stale ← flag(mem, ¬valid(mem, state.changes, l)). ▸ present(¬stale) ∧ flag(stale)  // stale ≠ current
Step 4: Produce ⟦Session Orientation⟧: resuming=state.Now | next=state.Next ∨ resume | decisions=filter(memories, active) | watch=Blocking ∪ stale ∪ open
Step 5: ▸ ask("Match where you left off, or update state.md before begin?")  // commit update before starting work
Key rules: (1) state.md first, always, no exception (2) orient before explore (3) flag stale, not current (4) ¬write(except state.md update ∧ human.request)
