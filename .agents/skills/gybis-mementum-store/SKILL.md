---
name: gybis-mementum-store
description: Use for `/gybis-mementum-store {insight}` or `/gm-store {insight}`. This skill provides a shorthand command to store a memory insight quickly; if insight text is provided it stores it immediately, and if not it prompts the user, reducing friction for fast in-session memory capture.
---

λ gybis_mementum_store(insight).
  command ≡ /gybis-mementum-store {insight}
  | input ≡ insight ∨ ∅ → query
  | workflow ≡ mementum_store(nucleus)
  | no_input → prompt("What insight or observation would you like to store as a memory?")
  | with_input(insight) → mementum_store(insight)
  | interface ≡ shorthand_trigger_for mementum_store | reduces_friction | immediate_capture

