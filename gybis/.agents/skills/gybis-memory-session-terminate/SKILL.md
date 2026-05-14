---
name: gybis-memory-session-terminate
description: This skill closes a session in a lossless, handoff-ready way: it records a complete state snapshot, stores any unstored memories, runs memory metabolization, drafts a dated session knowledge summary with human approval, and then performs a human-approved commit of mementum updates so work can resume cleanly in the next session.
---

λ(gybis-memory-session-terminate)
PURPOSE:persist(session)→resume_exact(¬loss[intent∪decisions∪threads])
PF:¬mem_store→err
S1:state.md←Σ{now∈progress(curr)∧¬planned,next=precise(action)⊢AI_start,blocking=∃preventing(fwdProg),recent=last(3‒5,actions)}|complete(state)≫brevity
S2:insight≡Σ{gate1=helps(futAI)∧rel(proj),gate2=effort(discovery)∧recur(lk),σ∈{insight,shift,decision,meta,mistake,win,pattern},tgt=mementum/memories/<slug>.md[≤200w],gate3=propose(human)⊢write}
S3:synthesize≡|mem(topic)|≥3∧¬knowPage(topic)→/gybis-memory-synthesize(topic)
S4:know[session-<date>]←Σ{what=concise(work),decisions=numbered(dec∪rat),threads=started∧¬fin∪q∧¬res,resume=exact(nextAct)[f|c|q],memSlugs=slugs(session)}|approve(human)⊢write
S5:δcommit≡git-add(mementum/)∧git-commit(🌀session-terminate <date>)|approve(human,msg)⊢exec
INV≡complete(state)∧¬placeholders(resume)∧∀wr→approve(human)∧|store|=0→explicit(¬store)∧fname=date(today)
μ≡S1⋅S2⋅S3⋅S4⋅S5
plain:given(→terminate)→flush(state)∧store(insights)∧synth(ready?)∥write(know)∥commit
