---
name: gybis-memory-recall
description: Use for `/gybis-memory-recall {topic}` or `/gm-recall {topic}`. This skill provides a shorthand command to recall memory by topic. If a topic is supplied, it runs recall for that topic; if not, it prompts the user to choose one, making memory lookup faster and more guided.
---

λ(gybis-memory-recall)
PURPOSE:given(topic)→Σ(knowledge)∪flag(stale)
PF:¬mem_store→err
S1:ΔR←git.grep(-i,topic,--mem/);//semantic—raw matches
S2:ΔT←git.log(-5,--oneline,--mem/,know/);//temporal—recency signal
S3:⊙type(topic)→sym→ΔF←git.grep(sym,--mem/);//🎯❌🔁💡—categorized
S4:ΔM←read(match(ΣR∪F));▸out:known(topic)|rels(dec)|stale←flag(M,¬valid(M,s.md,T))//≠current
S5:⊙|M[topic]|≥3∧¬know(topic)→▸suggest(synthesize);//self-improving loop
orient_vs_recall:orient=session_start(resume_context)∥recall=on_demand(query_answer)
R:(1)prior_syn≫re_derive(2)flag(stale)≡¬present(3)fib:1→2→3→5→8,depth₀=5(4)S5→auto_consolidate
plain:given(topic)→dig(doc)∧filter(rel)∧check(stale)→answer(known|risk|consolidate?)
