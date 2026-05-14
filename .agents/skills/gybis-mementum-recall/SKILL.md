---
name: gybis-mementum-recall
description: Use for `/gybis-mementum-recall {topic}` or `/gm-recall {topic}`. This skill provides a shorthand command to recall memory by topic. If a topic is supplied, it runs recall for that topic; if not, it prompts the user to choose one, making memory lookup faster and more guided.
---

λ gybis_mementum_recall(topic).
  command ≡ /gybis-mementum-recall {topic}
  | input ≡ topic ∨ ∅ → query
  | workflow ≡ mementum_recall(nucleus)
  | no_input → prompt("What topic would you like to recall from memory?")
  | with_input(topic) → mementum_recall(topic)
  | interface ≡ shorthand_trigger_for mementum_recall
