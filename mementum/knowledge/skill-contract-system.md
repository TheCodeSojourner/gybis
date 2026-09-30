---
type: Design
title: Skill Contract System
description: >
  How gybis skills declare their contract (kind, interaction, loop role, blocks)
  and how internal/gybis-skill-contract-check mechanically enforces it.
status: open
related: mementum/memories/skill-contract-checker-rationale.md, mementum/memories/contract-taxonomy-restraint.md, mementum/memories/interaction-mode-taxonomy.md, mementum/memories/mode-naming-conventions.md, mementum/memories/check-boundary-verifier-exception.md
---

# Skill Contract System

## Principle

Build the enforcement mechanism before the taxonomy it polices. A field with no
checker is unenforceable debt — drift is silent until a manual audit finds it.
Promote a value to a field and add its check in the same change.

Declare only what is enforced:

- `kind` earns a field — the only dimension invisible from the body.
- `mutates`, `verifier`, and read/write authority are *derived* from
  `tool_guard` / `_verify` / `_boundaries`; declaring them would create a second
  source of truth that can disagree with the first.
- `layer` is not a field — the skill name already encodes it.

## Declaration vocabulary

| Axis          | Values                                                                      | Where            |
| ------------- | --------------------------------------------------------------------------- | ---------------- |
| `kind`        | `domain` \| `memory` \| `help`                                              | frontmatter      |
| `interaction` | `autonomous` \| `interactive`                                               | purpose block    |
| `output_mode` | `response_only` \| `prompted_file_only` \| `default_file_only`              | describe/explain |
| `_loop_role`  | `loop_entry` \| `gather_context` \| `take_action` \| `verify` \| `maintain` | looping skills   |

Retired interaction tokens: `ai`, `auto`, `mixed`, `auto_polish`, `supervised`.

## The checker

`internal/gybis-skill-contract-check` verifies, per skill:

- declared `kind` against the name-derived class
- required header fields per kind (`purpose`, `input`, `output`, `interaction`,
  `gate` for domain)
- `interaction` ∈ canonical set, and any mode gate admits no forbidden mode
- `_loop_role` ∈ canonical roles, with the loops reference read
- every weeder verifies each artifact it writes
- create-only skills carry an absence guard (`¬∃` or `∅`)
- a reference preflight implies an actual reference read
- boundary blocks present — `_boundary` for describe/explain, otherwise
  `_boundaries` + `_regression_contract`
- absence spelling is `¬∃`/`∅`, never verbal `exists`

## Check vs act

`check` diagnoses; `refine`/`tend`/`weed` act. `spec-check` is the sole repair
exception, because it has a deterministic external oracle (`gybis-allium-gate`)
it must satisfy before COMPLETE. The other checks make semantic judgements and
stay read-only, handing off to the acting skills.
