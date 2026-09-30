---
type: Design
title: Weed Validity Gating
description: >
  Completion gating for the four weeders: each verifies the validity of whatever
  it actually wrote, not just that divergences reached zero.
status: open
related: mementum/memories/vocab-weed-validity-gate.md, mementum/memories/weed-validity-gating-complete.md, mementum/memories/conditional-verification-scope.md, mementum/memories/check-boundary-verifier-exception.md
---

# Weed Validity Gating

## Problem

A weeder resolves divergences across layers by rewriting artifacts. Verifying
only `remaining_divergences = ∅` lets a rewrite that broke Allium syntax, VSM
coherence, or tests still report COMPLETE.

## Mechanism

- `modified_artifacts ≔ ∅` at startup; every write branch records its artifact.
- `_verify` runs checks keyed to what was modified:

| Modified                        | Required                   |
| ------------------------------- | -------------------------- |
| `specs/`                        | `gybis-allium-gate = true` |
| `architecture.md`               | `vsm_coherence = true`     |
| `implementation` / `test_paths` | `test_suite_passes = true` |

- Completion requires `all_conditional_checks_pass = true` in the state machine
  and the regression contract.

## Conditional vs unconditional

- `arch-weed` and `spec-weed` verify **unconditionally** — they cannot run
  without those artifacts existing (spec-weed adds coverage + strict_coverage +
  tests).
- `req-weed` and `vocab-weed` verify **conditionally** on what they modified, so
  a vocab-only run does not fail merely because `specs/` exists untouched.

## Deadlock avoidance

If a conditional check fails but zero divergences remain, looping to
IDENTIFYING_DIVERGENCES would stall — a broken artifact is not resolvable by
divergence resolution. That case halts, names the failed artifact, and routes to
the owning repair skill (`spec-check` / `spec-tend` / `arch-tend`).
