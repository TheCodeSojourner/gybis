---
name: gybis-mementum-synthesize
description: This skill provides a fast shorthand command (/gybis-mementum-synthesize) that invokes the core memory synthesis workflow, making it easy to generate synthesized memory output without running the longer nucleus workflow manually.
---

λ gybis_mementum_synthesize().
  command ≡ /gybis-mementum-synthesize
  | workflow ≡ mementum_synthesize(nucleus)
  | interface ≡ shorthand_trigger_for mementum_synthesize
