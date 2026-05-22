---
name: gybis-memory-store
description: Use for `/gybis-memory-store {insight}` or `/gm-store {insight}`.
---

λ(gybis-memory-store)
PURPOSE:given(insight)→Σ(memory)∪commit
PF:¬insight→err
S1:gate1=helps(futAI)∧rel(proj)∧gate2=effort(discovery)∧recur(lk)|both→proceed∥fail→explain∧ask(store?)
S2:propose(σ∈{insight,shift,decision,meta,mistake,win,pattern}∧slug=kebab(id)∧content=<200w∧σ prefix)
S3:approve(human,σ,slug,content)⊢write(mementum/memories/{slug}.md)∧git-add(σ/memories/)∧git-commit(σ {slug})
S4:state.md←append(Recent,newMemory(σ {slug}))∧commit
INV≡|content|<200∧σ∈symbols∥∀write→approve(human)∥store(uncertain)∨¬store
μ≡S1⋅S2⋅S3⋅S4
plain:given(insight)→gate(both)∥propose(σ,slug,content)∥approve(human)∥write∥commit∥state.md