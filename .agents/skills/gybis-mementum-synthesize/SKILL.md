---
name: gybis-mementum-synthesize
description: Use for `/gybis-mementum-synthesize` or `/gm-synthesize`. This skill provides a fast shorthand command that invokes the core memory synthesis workflow, making it easy to generate synthesized memory output without running the longer nucleus workflow manually.
---

λ gybis_mementum_synthesize().
  command ≡ /gybis-mementum-synthesize
  | workflow ≡ mementum_synthesize(nucleus)
  | interface ≡ shorthand_trigger_for mementum_synthesize
