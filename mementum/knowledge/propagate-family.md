---
type: Design
title: Propagate Family
description: >
  The X-propagate skills bootstrap the layer immediately below X (greenfield);
  that layer's tend/refine/weed skills then own it.
status: open
related: mementum/memories/propagate-seed-then-own.md, mementum/memories/req-propagate-write-authority.md, mementum/memories/distill-input-direction.md
---

# Propagate Family

## Seed then own

Propagation is a bootstrap, not an authority. `X-propagate` creates the initial
artifact of the layer below `X`; the layer's own `tend`/`refine`/`weed` skills
own it thereafter. Vocabulary and architecture stay developer-owned durable
artifacts — propagation only gets the project off the blank page.

Why not pure derivation: if `req → vocab` were mechanical, vocabulary would
restate requirements and lose the independent authority that makes it a
constraint layer. Under seed-then-own each layer is authored against its own
concerns (vocab = domain language agreement; arch = VSM structure) and stays
more durable than the layer below.

## Convention

| Skill             | Creates                                                  |
| ----------------- | -------------------------------------------------------- |
| `req-propagate`   | req → vocabulary (`vocabulary.md` seeded from REQ terms) |
| `vocab-propagate` | vocab → architecture                                     |
| `arch-propagate`  | arch → specs                                             |
| `spec-propagate`  | specs → code/tests                                       |

Greenfield chain: `/gybis-req-elicit` → `/gybis-req-propagate` →
`/gybis-vocab-propagate` → `/gybis-arch-propagate` → `/gybis-spec-propagate`.

## Direction symmetry

Forward (greenfield) is top-down and bootstrapped by the propagate family.
Brownfield is strictly bottom-up: each layer is distilled from the layers below
it, never from a layer above (`spec-distill` → `arch-distill` → `vocab-distill`
→ `req-distill`).

`vocab-tend` is deliberately **not** a bootstrap step — its gate requires
`vocabulary.md ∃`, so it can only own an existing vocabulary.

## Write authority

`req-propagate` writes `vocabulary.md` when absent (seeded `status: draft`,
`seeded_from: requirements/`) and appends terms only when present. The invariant
narrows from "only vocab skills write `vocabulary.md`" to "vocab skills own
`vocabulary.md` after bootstrap".
