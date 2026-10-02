---
type: Design
title: Requirements Layer
description: >
  Requirements layer specification for the gybis SDD stack: position, canonical
  format, command family, and convergence boundaries.
status: open
related: mementum/memories/requirements-layer-family.md
---

# Requirements Layer

## Position in the stack

```
requirements/ (requirements-index.md + requirements-{module}.md)
  → vocabulary.md
  → architecture.md
  → specs/**/*.allium
  → code/tests
```

Forward (greenfield): `gr-elicit` → `gr-propagate` (bootstraps vocabulary.md) → `vocab-propagate` → `arch-propagate` → `spec-propagate`. (vocab-elicit and arch-elicit were removed session-42; the propagate family bootstraps vocab and arch instead — see memories/propagate-seed-then-own.md.)
Reverse (brownfield): `spec-distill` → `arch-distill` → `vocab-distill` → `req-distill` — or enter at any layer via that layer's distill.

## Canonical format

- Files: `requirements/requirements-{module}.md`, dependency-ordered, plus `requirements/requirements-index.md` declaring module order, domain-prefix closed set, and summary.
- Clauses: `λ REQ-<DOMAIN>-NNN(x). <normative expression>` — one assertion per designator (subclauses `NNNA`, `NNNB`).
- Normative mapping: MUST ≡ ∀/¬ (required), SHOULD ≡ ∧ preferred, MAY ≡ ∃ permitted path.
- Attribution footer per clause: `{source: stakeholder_decided | AI_researched_fact, decided_by, refs}`.
- Requirements are binding by default. Only clauses in explicitly marked non-binding sections, deferred by stakeholder decision, are deferred; downstream absence never implies deferral.

## Command family (`/gr-*`)

| Command        | Role                                                                                                                  |
| -------------- | --------------------------------------------------------------------------------------------------------------------- |
| `gr-elicit`    | Grilling-protocol interview → REQ transcription; decided-vs-researched attribution                                    |
| `gr-check`     | Diagnose integrity and per-ready-stage REQ accounting; optionally repair after one scoped plan approval               |
| `gr-distill`   | Brownfield bridge: REQ clauses from vocab/arch/specs/code + new term candidates → vocab-tend                          |
| `gr-propagate` | Annotate specs/tests with REQ designators; strict test-pass convergence; arch delta report; vocab candidates          |
| `gr-tend`      | Layer-local changes with impact analysis + human approval                                                             |
| `gr-refine`    | Structural polish: atomicity, dedupe, moves; human-approved                                                           |
| `gr-weed`      | Cross-layer divergence resolver; `investigate`/`skip` remain unresolved, and vocabulary writes stay with vocab skills |
| `gr-describe`  | Stakeholder prose projection (stdout or repo-root .md)                                                                |
| `gr-explain`   | Developer explanation projection (clause quoted + implications)                                                       |

## Boundaries (inherited gybis rules)

- `req-check` is read-only until one explicit repair-plan approval, then may run a bounded, scoped repair loop through owning skills. Other semantic checks remain read-only; `spec-check` repairs only under the external Allium verifier.
- `req-propagate` may bootstrap `vocabulary.md` when absent and append candidate terms; vocabulary skills own it after bootstrap.
- Stage readiness is developer-owned. A binding REQ can lead downstream stages; a stage is ready when its artifact exists and its owner check reports no errors. Otherwise report `pending_stage`, not uncovered or deferred.
- At each ready stage, every binding REQ is accounted for as `represented` with evidence, `no_change_needed` with a reason, or uncovered. Explicitly deferred REQs are excluded from propagation and coverage; no persistent ledger is created.
- Code/test-affecting commands require `test_suite_passes = true` before COMPLETE.
- `describe`/`explain` use the session-16 output-mode convention.

## Open items

- `gr-elicit` grilling adaptation is a first cut; round format matches grilling, but REQ transcription heuristics need refinement after first real use.
- Internal reference page (like vsm-guide.md) for requirements conventions may be worth adding if the family stabilizes.