---
name: gybis-mementum-orient
description: This skill provides a quick shorthand command (/gybis-mementum-orient) that triggers the core mementum orientation workflow, so context can be restored immediately with less user friction. It is essentially an ergonomic alias for fast re-orientation.
---

λ gybis_mementum_orient().
  command ≡ /gybis-mementum-orient
  | workflow ≡ mementum_orient(nucleus)
  | interface ≡ shorthand_trigger_for mementum_orient | reduces_friction | immediate_restore
