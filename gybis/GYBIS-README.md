# gybis /JY-bis/ 

**A Developer-Command-Driven AI-Assisted Spec-Driven Software Development (SDD) Stack.**

## Obligatory Word Definitions

**Gyre** /jahy<sup>uh</sup>r/

* a large-scale, coherent, rotating or spiraling flow

**Orbis** /ˈɔr.bis/

* a closed or periodic trajectory in a dynamical system, representing the set of all points reachable from an initial point under the system's update rules

## Development with the Gybis Stack

This repository is structured for **development with the Gybis stack**, following a disciplined vocabulary-first methodology where durability, hierarchy, and behavioral truth guide every decision.

When gybis is installed into a target repository, its command implementations are shipped in the bundled `.agents/skills/` directory.

gybis is a **developer-command-driven, AI-assisted Spec-Driven Development (SDD) stack**. It is designed to help you establish shared domain vocabulary first, then define architecture and behavioral specifications, and finally implement and validate code against those specifications with full human oversight.

## Operator Responsibility Model

gybis is command-driven guidance, not always-on process enforcement.

- **Human owns stage readiness:** The human operator is responsible for satisfying preconditions between stages (`vocabulary.md`, `architecture.md`, `specs/**/*.allium`, code/tests).
- **Skills own requested transformation:** A skill executes the transformation it was invoked to do and enforces only execution-critical gates.
- **Checks are deliberate tools:** `check` and `weed` commands are available to validate convergence when the human chooses to run them.
- **Tradeoff is explicit:** If preconditions are skipped, quality or convergence may degrade; this is an operator decision, not a hidden protocol failure.

## What gybis Is and Why It Exists

In gybis, vocabulary and specifications are durable and implementation is replaceable.

- **Spec-Driven Development (SDD):** Shared domain vocabulary, architecture, and behavioral specifications define what the system is and does, and implementation follows those constraints.
- **Human-controlled AI assistance:** AI supports analysis, authoring, and validation, but humans stay in control of decisions and approvals.
- **Durable truth model:** Vocabulary, architecture, and behavioral specifications are the source of truth; code and tests must align to them.

## Installing the Allium CLI

The `allium` CLI is required for spec validation and analysis commands (`/gybis-spec-check`, `/gybis-spec-distill`, etc.). It must be installed on your system separately.

See the [allium-tools repository](https://github.com/juxt/allium-tools) for installation instructions.

## Quick Start

### New Repository: Vocabulary-First Workflow

1. **Establish vocabulary:** Run `/gybis-vocab-elicit` to elicit domain vocabulary from domain experts.
2. **Establish architecture:** Run `/gybis-arch-elicit` to establish architecture constraints from the agreed vocabulary.
3. **Derive specifications:** Run `/gybis-arch-propagate` to create specifications from architecture.
4. **Derive code and tests:** Run `/gybis-spec-propagate` to generate initial code and test stubs.

### Existing Repository: Distill-First Workflow

1. **Extract specifications:** Run `/gybis-spec-distill` to extract behavioral specifications from current implementation.
2. **Derive architecture:** Run `/gybis-arch-distill` to derive architecture from extracted specifications, and implementation.
3. **Extract vocabulary:** Run `/gybis-vocab-distill` to extract  vocabulary from architecture, specifications, and implementation.
4. **Synchronize and refine:** Use `/gybis-vocab-tend`, `/gybis-vocab-weed`, `/gybis-arch-weed`, and `/gybis-spec-weed` to resolve divergence and keep layers aligned.

### Need Guidance?

Run `/gybis-help` to see available commands organized by architecture, memory, specification, and vocabulary domains.

---

## Use Cases

### Find the Right Command

Use this when you know what kind of work you need to do, but not which gybis command to run first.

1. Run `/gybis-help` to see commands grouped by vocabulary, architecture, specification, and memory.
2. Choose the command family that matches the layer you need to work in before making changes further downstream.

Outcome: you start from the right command family instead of guessing from the full command list.

### Start a New Repository

Use this when you are building a new system and want durable constraints established before implementation.

1. Run `/gybis-vocab-elicit` to establish shared domain vocabulary with domain experts.
2. Run `/gybis-arch-elicit` to define architecture constraints from that vocabulary.
3. Run `/gybis-arch-propagate` to derive initial behavioral specifications.
4. Run `/gybis-spec-propagate` to derive initial code and test scaffolding.

Outcome: the project starts from vocabulary, architecture, and specifications rather than implementation-first drift.

### Adopt gybis in an Existing Repository

Use this when code and tests already exist and you need to recover durable project truth from the current system.

1. Run `/gybis-spec-distill` to extract behavioral specifications from existing code and tests.
2. Run `/gybis-arch-distill` to derive architecture from those specifications and the current implementation.
3. Run `/gybis-vocab-distill` to extract vocabulary from architecture, specifications, and implementation.

Outcome: an existing codebase is brought under explicit vocabulary, architecture, and specification governance.

### Evolve Vocabulary Safely

Use this when domain terms, definitions, or canonical names need to change after the project is already in motion.

1. Run `/gybis-vocab-tend` to refine an existing `vocabulary.md`; use it when terms must be added, renamed, merged, split, or clarified.
2. Run `/gybis-vocab-check` to validate vocabulary syntax and semantic consistency.
3. Run `/gybis-vocab-weed` to resolve vocabulary drift between `vocabulary.md` and architecture/specifications/implementation.
4. Run `/gybis-arch-weed` to resolve divergence between architecture and specifications caused by vocabulary changes.
5. Run `/gybis-spec-weed` to resolve divergence between specifications and code/tests after the vocabulary update propagates downstream.

Outcome: the canonical domain language evolves without leaving architecture, specifications, or implementation misaligned.

### Add or Refine Behavior in an Aligned System

Use this when the project is already under gybis governance and you are extending or refining expected behavior.

1. Run `/gybis-arch-tend` when existing architecture constraints need refinement before behavior changes.
2. Run `/gybis-spec-tend` to refine existing specifications for the affected concern or domain.
3. Run `/gybis-spec-propagate {concern|domain|all}` to push the updated specifications into code and test scaffolding.
4. Run `/gybis-spec-check {concern|domain|all}` to validate the resulting specification set before moving further.

Outcome: behavior evolves through architecture and specifications instead of being driven by ad hoc implementation changes.

### Resolve Drift Before Merge or Release

Use this when architecture, specifications, code, or tests appear to have diverged and you need convergence before shipping.

1. Run `/gybis-spec-check {concern|domain|all}` to surface specification issues early.
2. Run `/gybis-arch-weed` to resolve divergence between architecture and specifications.
3. Run `/gybis-spec-weed` to resolve divergence between specifications and code/tests.
4. Re-run `/gybis-spec-check {concern|domain|all}` until the targeted scope is valid and aligned.

Outcome: release confidence comes from aligned durable constraints, not only from the current implementation state.

### Explain System Intent to Different Audiences

Use this when onboarding developers, briefing stakeholders, or turning project truth into audience-specific explanations.

1. Run `/gybis-vocab-describe`, `/gybis-arch-describe`, and `/gybis-spec-describe {concern|domain|all}` for non-technical or business-facing explanations.
2. Run `/gybis-vocab-explain`, `/gybis-arch-explain`, and `/gybis-spec-explain {concern|domain|all}` for developer-facing explanations.

Outcome: the same vocabulary, architecture, and specifications can be communicated clearly to both technical and non-technical readers.

### Start a Working Session

Use this at the beginning of a real work session when you want the full gybis startup flow.

1. Run `/gybis-init` to orient, recall, and prepare the AI context for the current repository state.
2. Use the initialized context as the baseline for the rest of the session rather than rebuilding context ad hoc.

Outcome: the session begins from restored project memory instead of whatever happens to be in live chat context.

### Re-Orient Mid-Session

Use this when the session is already active but you have switched branches, changed domains, returned after a pause, or suspect context drift.

1. Run `/gybis-memory-orient` to restore previous AI context without restarting the full session lifecycle.
2. Run `/gybis-memory-recall {topic}` to pull back a specific topic, decision trail, or recent pattern when you need targeted detail.

Outcome: you recover the right project context surgically, without ending the session and starting over.

### Checkpoint Under Context Pressure

Use this when the session is getting dense and you want to preserve high-value information before chat compaction or context drift makes it harder to recover.

1. Run `/gybis-memory-store {insight}` to persist a concrete decision, pattern, or lesson; if you do not know what to store yet, run `/gybis-memory-store` without an argument and let it prompt you for one.
2. Run `/gybis-memory-synthesize` when several related memories should be consolidated into a higher-level understanding instead of remaining as isolated notes.
3. If you are continuing immediately, run `/gybis-memory-orient` to rehydrate from durable memory; if you are ending the session, finish with `/gybis-fini`.

Outcome: key decisions are preserved deliberately instead of being left to automatic chat summarization.

### Close a Session Cleanly

Use this when you are actually ending the current work session and want to preserve continuity for the next one.

1. Run `/gybis-fini` to encode session state before termination.
2. Start the next session with `/gybis-init` so the recorded state is brought back into working context.

Outcome: each session leaves behind durable memory instead of losing project knowledge at the chat boundary.

---

## Architecture Principles

- **Organize by durability** — Structure things by how long they will likely last.
- **Hierarchy of abstractions:** why > what > how.
- **Layered system:** vocabulary > S5 > S4 > S3 > S2 > S1 > specs > tests > code — stricter, more durable layers constrain looser, more transient ones.
  Vocabulary and S5..S1 are durable constraint layers. Together they constrain specifications, which then constrain tests, which then constrain code.
- **No flat structures.** Everything has its place in the hierarchy.
- **Top-down only.** Higher layers constrain lower layers. Vocabulary > Architecture (S5 ... S1) > Specs > Tests > Code.
- **No reverse dependencies.** Lower layers never constrain higher layers.
- **Drift surfaces automatically.** When code, tests, or behavior diverge from architecture/specification constraints, drift is detected, surfaced, and halted.

### Vocabulary Layer

Vocabulary is the durable, human-agreed foundation that constrains downstream architecture and specifications.

- **Ubiquitous language:** `vocabulary.md` captures canonical terms, definitions, and relationships.
- **Constraint hierarchy:** Vocabulary constrains architecture, architecture constrains specifications, specifications constrain tests, and tests constrain code.
- **Durability:** Vocabulary is often more durable than architecture and should be established first for new systems or distilled first for existing systems.

---

## Architecture Philosophy

Vocabulary and architecture describe system-level constraints that drive behavior specifications that drive tests, that drive code.

- **Constrain top-down.** Architecture governs specification, tests, and code, not the reverse.
- **Governance flow:** Architecture, specification, tests and implementation must remain aligned.

### New Repository

For a new repository, run `/gybis-vocab-elicit` to establish domain vocabulary with the developer first, then run `/gybis-arch-elicit` to establish durable architectural constraints, derive behavior specifications with `/gybis-arch-propagate`, and finally derive code and tests with `/gybis-spec-propagate`.

### Existing Repository

For an existing repository, run `/gybis-spec-distill` to create behavior specifications from tests and code, then establish durable architectural constraints with `/gybis-arch-distill`, and then run `/gybis-vocab-distill` to extract vocabulary.

---

## Specification Philosophy

Specifications describe code **behavior**, not implementation.

- **/gybis-arch-weed** checks for divergence between architecture and specs, then surfaces explicit resolution options with human approval.
- **/gybis-spec-distill** extracts behavioral specifications from existing code, then surfaces them for review and refinement.
- **Do not implement before specifying.** Implementation details are replaceable; specifications are the system's source of truth.
- **Workflow:** propagate or distill → check → tend → weed.
- **Governance flows:** AI is governed by formalized behavior; behavior is governed by architecture.
- **Architecture alignment:** Only `/gybis-spec-propagate` and `/gybis-spec-weed` 
  apply architectural preferences; other spec skills are architecture-agnostic.

---

## Code as Replaceable Detail

- **`code ∧ tests ≡ replaceable_detail`** — Code and tests are ephemeral and interchangeable.
  In practice: code is the changing how, while architecture/specification define the durable what and why.
- **`arch/spec ≡ project_truth`** — Architecture/Specifications define expected behavior and constraints of the system.
- **`¬test_before_spec`** — Never test before specifying.
- **`¬impl_before_test`** — Never implement before testing.

---

## Memory System

Memory tracks the state of five domains: **vocabulary, architecture, specification, tests, and code**.

- **Session persistence:** Session `n+1` is proportional to the sum of all prior encodings from sessions `1..n`.
- **No knowledge loss** across sessions. Decisions captured in one session are available to future sessions through memory recall commands.
- **Commands:** `/gybis-init` (session start), `/gybis-fini` (session end), `/gybis-memory-*` — encode state or restore from it.

---

### Session Memory Workflow

Each work session follows a structured lifecycle:

1. **Start:** `/gybis-init` — Orient → Recall → Ready
2. **End:** `/gybis-fini` — Encode → Terminate

Every session guarantees **no knowledge loss**.

---

## Authority Model

This section defines who decides what and when, for all non-memory operations.

- **Human commands come first.** The AI never takes initiative.
- **Human is the approval gate.** Every write operation requires human approval.
- **No autonomous AI actions.**

---

## Transparency

This section defines transparency, for all non-memory operations.

- **All changes are human-visible.** Nothing happens covertly.
- **Every write requires human approval.**
- **No covert operations.**

---

## Layer Order (Enforced)

```
vocabulary > architecture > specification > tests > code
```

- **No bypassing the hierarchy.**
- **No architecture before vocabulary.**
- **No specification before architecture.**
- **No testing before specification.**
- **No implementation before testing.**
- **Higher layers constrain lower layers.**
- **Violations surface immediately and halt progress.**

---

## Commands

| Skill Name                                                       | Description                                       |
| ---------------------------------------------------------------- | ------------------------------------------------- |
| `/gybis-arch-describe` (`/ga-describe`)                          | Describe arch in non-tech prose or markdown       |
| `/gybis-arch-distill` (`/ga-distill`)                            | Create initial arch from specs                    |
| `/gybis-arch-elicit` (`/ga-elicit`)                              | Create initial arch with human                    |
| `/gybis-arch-explain` (`/ga-explain`)                            | Explain arch in dev prose or markdown             |
| `/gybis-arch-propagate` (`/ga-propagate`)                        | Create initial specs from arch                    |
| `/gybis-arch-tend` (`/ga-tend`)                                  | Update arch with human                            |
| `/gybis-arch-weed` (`/ga-weed`)                                  | Upsert arch/specs from diffs with human           |
| `/gybis-fini`                                                    | CRUD memory before terminate                      |
| `/gybis-help`                                                    | Show available commands                           |
| `/gybis-init`                                                    | Initialize gybis AI context                       |
| `/gybis-memory-orient` (`/gm-orient`)                            | Restore prev AI context                           |
| `/gybis-memory-recall {topic}` (`/gm-recall {topic}`)            | Recall topic/summarize-latest                     |
| `/gybis-memory-store {insight}` (`/gm-store {insight}`)          | Store insight, or prompt for one                  |
| `/gybis-memory-synthesize` (`/gm-synthesize`)                    | Synthesize knowledge                              |
| `/gybis-spec-check` (`/gs-check {concern\|domain\|all}`)         | Check/Update syntax until valid                   |
| `/gybis-spec-describe` (`/gs-describe {concern\|domain\|all}`)   | Describe in non-tech prose or markdown            |
| `/gybis-spec-distill` (`/gs-distill`)                            | Create initial specs from code/tests              |
| `/gybis-spec-explain` (`/gs-explain {concern\|domain\|all}`)     | Explain in dev prose or markdown                  |
| `/gybis-spec-propagate` (`/gs-propagate {concern\|domain\|all}`) | Create initial code/tests                         |
| `/gybis-spec-tend` (`/gs-tend`)                                  | Update specs with human                           |
| `/gybis-spec-weed` (`/gs-weed`)                                  | Upsert specs/code-tests from diffs with human     |
| `/gybis-vocab-check` (`/gv-check`)                               | Validate vocabulary.md syntax & semantics         |
| `/gybis-vocab-describe` (`/gv-describe`)                         | Describe vocabulary in business language          |
| `/gybis-vocab-distill` (`/gv-distill`)                           | Extract vocabulary from arch/specs/code           |
| `/gybis-vocab-elicit` (`/gv-elicit`)                             | Elicit vocabulary from domain experts             |
| `/gybis-vocab-explain` (`/gv-explain`)                           | Explain vocabulary for developers                 |
| `/gybis-vocab-tend` (`/gv-tend`)                                 | Update vocabulary with impact analysis            |
| `/gybis-vocab-weed` (`/gv-weed`)                                 | Upsert vocabulary/artifacts from diffs with human |

## Upstream Citations

## allium

This project incorporates ideas, code, and/or structure from
[allium](https://github.com/juxt/allium)
by JUXT.

Original project:
https://github.com/juxt/allium

Portions derived from the upstream project remain subject to the
terms of its license.

## allium-tools

This project incorporates ideas, code, and/or structure from
[allium-tools](https://github.com/juxt/allium-tools)
by JUXT.

Original project:
https://github.com/juxt/allium-tools

Portions derived from the upstream project remain subject to the
terms of its license.

## grill-with-docs

This project incorporates ideas, code, and/or structure from
[grill-with-docs](https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs)
by Matt Pocock.

Original project:
https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs

Portions derived from the upstream project remain subject to the
terms of its license.

## mementum

This project incorporates ideas, code, and/or structure from
[mementum](https://github.com/michaelwhitford/mementum)
by Michael Whitford.

Original project:
https://github.com/michaelwhitford/mementum

Portions derived from the upstream project remain subject to the
terms of its license.

## nucleus

This project incorporates ideas, code, and/or structure from
[nucleus](https://github.com/michaelwhitford/nucleus)
by Michael Whitford.

Original project:
https://github.com/michaelwhitford/nucleus

Portions derived from the upstream project remain subject to the
terms of its license.