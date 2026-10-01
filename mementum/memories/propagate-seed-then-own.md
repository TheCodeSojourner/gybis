---
type: Decision
symbol: 🎯
title: Propagate bootstraps a layer; tend/refine/weed then own it
related: distill-input-direction, requirements-layer-family, req-propagate-write-authority
---

Propagate bootstraps a layer; tend/refine/weed then own it.

Propagation is a bootstrap, not an authority. A `*-propagate` skill creates the initial artifact of the layer below it; that layer's own `tend`/`refine`/`weed` skills own it thereafter. Vocab and architecture remain developer-owned durable artifacts — propagation only gets the project off the blank page.

Why not pure derivation: if `req → vocab` were purely mechanical, vocabulary would restate requirements and lose the independent authority that makes it a constraint layer. Under seed-then-own, each layer is authored against its own concerns (vocab = domain language agreement; arch = VSM structure) and stays more durable than the layer below.

Greenfield chain: `/gybis-req-elicit` → `/gybis-req-propagate` → `/gybis-vocab-propagate` → `/gybis-arch-propagate` → `/gybis-spec-propagate`.

Note: `gybis-vocab-tend` is deliberately NOT a bootstrap step — its gate requires `vocabulary.md ∃`, so it can only own an existing vocabulary, never create one. `tend`/`refine`/`weed` own a layer; they do not bootstrap it.
