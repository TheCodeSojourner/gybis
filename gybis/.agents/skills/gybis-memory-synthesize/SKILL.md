---
name: gybis-memory-synthesize
description: This skill provides a fast shorthand command (/gybis-memory-synthesize) that invokes the core memory synthesis workflow, making it easy to generate synthesized memory output without running the longer nucleus workflow manually.
---

λ(gybis-memory-synthesize)
PURPOSE:given(topic?)→Σ(knowledgePage)∪commit∪loop
PF:¬memStore→err
S0:topic?|specified→S1∥¬specified→discover(Σtopics←unique(topicsFrom:mementum/memories/))∧filter(|mems|≥3)∧present(Σcand)∧get(approvedTopics)
S1:Σmem←git-grep(-i,topic,mementum/memories/)∧git-log(-n13,oneline,mementum/memories/)∧read(Σmem)
S2:kp=mementum/knowledge/{topic}.md|exists→read(kp)∧update∥¬exists→create
S3:kp←Σ{title:topicTitle,status:open|designing|active|done,category:arch|spec|memory|dev,tags:[...],related:[],depends-on:[],content:synthesized(AI→fut)}
S4:approve(human,kp)⊢git-add(mementum/knowledge/{topic}.md)∧git-commit(💡synthesize:{topic})
S5:remainingTopics|Σcand\{topic}→S0∥empty→halt
INV≡write(AI→fut)∥openOK≠complete∥kp=update(inPlace)∧mem=accumulate∥∀commit→approve(human)∥∀topic→approve(human)
μ≡S0?⋅S1⋅S2⋅S3⋅S4⋅S5
plain:given(topic?)→discover∧filter(|mems|≥3)∥present(candidates)∧getApproval∥loop(topic):gather∥checkKP∥draft∥approve∥commit∥next∥halt
