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
Reverse (brownfield): `spec-distill` → `arch-distill` → `vocab-distill` → `gr-distill` — or enter at any layer via that layer's distill.

## Canonical format

- Files: `requirements/requirements-{module}.md`, dependency-ordered, plus `requirements/requirements-index.md` declaring module order, domain-prefix closed set, and summary.
- Clauses: `λ REQ-<DOMAIN>-NNN(x). <normative expression>` — one assertion per designator (subclauses `NNNA`, `NNNB`).
- Normative mapping: MUST ≡ ∀/¬ (required), SHOULD ≡ ∧ preferred, MAY ≡ ∃ permitted path.
- Attribution footer per clause: `{source: stakeholder_decided | AI_researched_fact, decided_by, refs}`.
- Deferred future work lives in explicitly marked non-binding sections.

## Command family (`/gr-*`)

| Command        | Role                                                                                                                    |
| -------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `gr-elicit`    | Grilling-protocol interview → REQ transcription; decided-vs-researched attribution                                      |
| `gr-check`     | Read-only diagnostics: designators, ordering, clause well-formedness, coverage, traceability                            |
| `gr-distill`   | Brownfield bridge: REQ clauses from vocab/arch/specs/code + new term candidates → vocab-tend                            |
| `gr-propagate` | Annotate specs/tests with REQ designators; strict test-pass convergence; arch delta report; vocab candidates            |
| `gr-tend`      | Layer-local changes with impact analysis + human approval                                                               |
| `gr-refine`    | Structural polish: atomicity, dedupe, moves; human-approved                                                             |
| `gr-weed`      | Cross-layer divergence resolver (`req\|arch\|spec\|test\|propagate\|na\|investigate\|skip`); never writes vocabulary.md |
| `gr-describe`  | Stakeholder prose projection (stdout or repo-root .md)                                                                  |
| `gr-explain`   | Developer explanation projection (clause quoted + implications)                                                         |

## Boundaries (inherited gybis rules)

- `check` diagnoses only; resolution in `tend`/`weed` (arch-check boundary). Exception: `spec-check` may repair, because its corrections are verified by the external allium CLI — autonomous correction requires an external verifier.
- `vocabulary.md` is only written by vocabulary skills; req skills emit candidates.
- Stage readiness is developer-owned; missing requirements/ never hard-halts downstream skills.
- Code/test-affecting commands require `test_suite_passes = true` before COMPLETE.
- `describe`/`explain` use the session-16 output-mode convention.

## Open items

- `gr-elicit` grilling adaptation is a first cut; round format matches grilling, but REQ transcription heuristics need refinement after first real use.
- Domain prefix closed set (REQ-PLAT, REQ-VAL, REQ-FN, ...) needs a recommended default set in the index template.
- Internal reference page (like vsm-guide.md) for requirements conventions may be worth adding if the family stabilizes.