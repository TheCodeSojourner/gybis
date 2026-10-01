# gybis /JY-bis/ 

**A Developer-Command-Driven AI-Assisted Spec-Driven Software Development (SDD) Stack.**

## Obligatory Word Definitions

**Gyre** /jahy<sup>uh</sup>r/

* a large-scale, coherent, rotating or spiraling flow

**Orbis** /ˈɔr.bis/

* a closed or periodic trajectory in a dynamical system, representing the set of all points reachable from an initial point under the system's update rules

## Development with the Gybis Stack

This repository is structured for **development with the Gybis stack**, following a disciplined requirements-first methodology where durability, hierarchy, and behavioral truth guide every decision.

When gybis is installed into a target repository, its command implementations are shipped in the bundled `.agents/skills/` directory.

It is designed to help you establish requirements first, then define shared domain vocabulary, architecture, and behavioral specifications, and finally implement and validate code against those durable constraints with full human oversight.

## Installing the Allium CLI

The `allium` CLI is required for spec validation and analysis commands (`/gybis-spec-check`, `/gybis-spec-distill`, etc.). It must be installed on your system separately.

See the [allium-tools repository](https://github.com/juxt/allium-tools) for installation instructions.

## Quick Start

### New Repository: Requirements-First Workflow

1. **Establish requirements:** Run `/gybis-req-elicit` to elicit requirements from stakeholders via grilling interview rounds.
2. **Bootstrap vocabulary:** Run `/gybis-req-propagate` to derive initial `vocabulary.md` from the requirement terms, then validate it with `/gybis-vocab-check`.
3. **Bootstrap architecture:** Run `/gybis-vocab-propagate` to derive initial `architecture.md` from requirements and vocabulary, then validate it with `/gybis-arch-check`.
4. **Derive specifications:** Run `/gybis-arch-propagate` to create specifications from architecture, then validate them with `/gybis-spec-check`.
5. **Derive code and tests:** Run `/gybis-spec-propagate` to generate initial code and test stubs.

Each propagate bootstraps a layer; the layer's own `/gybis-*-tend`, `/gybis-*-refine`, and `/gybis-*-weed` skills own it from then on.

### Existing Repository: Distill-First Workflow

1. **Extract specifications:** Run `/gybis-spec-distill` to extract behavioral specifications from current implementation, then validate them with `/gybis-spec-check`.
2. **Derive architecture:** Run `/gybis-arch-distill` to derive architecture from extracted specifications and implementation, then validate it with `/gybis-arch-check`.
3. **Extract vocabulary:** Run `/gybis-vocab-distill` to extract vocabulary from architecture, specifications, and implementation, then validate it with `/gybis-vocab-check`.
4. **Extract requirements:** Run `/gybis-req-distill` to distill the initial requirement set from vocabulary, architecture, specifications, and implementation, then validate it with `/gybis-req-check`.

Each distill reconstructs a layer from the artifacts below it; the layer's own `/gybis-*-tend`, `/gybis-*-refine`, and `/gybis-*-weed` skills own it from then on.

### Need Guidance?

Run `/gybis-help` to see available commands, grouped by command family.

---

## What gybis Is and Why It Exists

In gybis, requirements, vocabulary, architecture, and specifications are durable and implementation is replaceable.

- **Spec-Driven Development (SDD):** Shared requirements, domain vocabulary, architecture, and behavioral specifications define what the system is and does, and implementation follows those constraints.
- **Human-controlled AI assistance:** AI supports analysis, authoring, and validation, but humans stay in control of decisions and approvals.
- **Durable truth model:** Requirements, vocabulary, architecture, and behavioral specifications are the source of truth; code and tests must align to them.

## Developer Responsibility Model

gybis is command-driven guidance, not always-on process enforcement.

- **Developer owns stage readiness:** The developer is responsible for satisfying preconditions between stages (`requirements/requirements-*.md`, `vocabulary.md`, `architecture.md`, `specs/**/*.allium`, code/tests).
- **Skills own requested transformation:** A skill executes the transformation it was invoked to do and enforces only execution-critical gates.
- **Checks are deliberate tools:** `check` and `weed` commands are available to validate convergence when the developer chooses to run them.
- **Tradeoff is explicit:** If preconditions are skipped, quality or convergence may degrade; this is a developer decision, not a hidden protocol failure.

## AI Is Nondeterministic: Expect to Rerun

If you are new to AI-assisted development, set this expectation early: **the same command, run twice against the same repository, can produce different results.** That is a property of the underlying AI model, not a defect in gybis or in your repository. Plan for iteration; do not expect one perfect pass.

Which commands you rerun, and why:

- **`check`, `describe`, `explain` — rerun freely.** These never change your requirements, vocabulary, architecture, specs, or code. `check` is purely diagnostic; `describe`/`explain` only read a layer and may emit a separate explainer document that you name. Run them as often as you like.
- **`distill`, `propagate` — run once per layer.** These are bootstrap commands, gated on the target artifact *not* existing. Once the artifact exists, rerunning them halts by design (`already exist` / `not found`) instead of overwriting your work.
- **`refine`, `tend`, `weed` — rerun until converged.** These are the iterative loops. To improve an artifact that already exists, cycle `check → refine → tend → weed` as many times as needed; each pass moves the artifact closer to its intended state.

How to tell a rerun helped:

- Compare `check` findings before and after; fewer issues is progress.
- An artifact has **stabilized** when `check` passes and a fresh run produces no further change.
- Git is your safety net: review the diff or commit between runs, so any run you dislike can be discarded.

Rule of thumb: treat an AI result as a **draft to converge**, not a finished artifact to accept. Running a skill again — or running a different skill on the same layer — is normal workflow, not a sign that something went wrong.

## Check, Refine, Tend, and Weed Philosophy

The gybis workflow is built around four deliberate actions that the human chooses at the right time:

- `check` tells you whether a layer is valid and what kind of drift or inconsistency is present.
- `refine` polishes a single layer's structure and readability while preserving intended meaning.
- `tend` evolves a single layer with human-approved intent before the drift spreads downstream.
- `weed` resolves mismatch across neighboring layers or implementation when two artifacts no longer describe the same truth.

The boundary is: **`check` diagnoses; `refine`, `tend`, and `weed` act.** One exception is `spec-check`, which also repairs `.allium` errors directly; the other check skills only diagnose and hand off to the acting skills.

The pattern is intentionally hierarchical: run `check` first, and its structural or clarity findings nominate `refine`; `tend` applies intended meaning changes, and `weed` reconciles layers that no longer agree. The human decides the scope and approves any write.

| Operation | When to use it                                    | Human role                                                     | Typical outcome                    |
| --------- | ------------------------------------------------- | -------------------------------------------------------------- | ---------------------------------- |
| `check`   | You want a diagnostic pass before making changes  | Review the report and decide whether the layer needs attention | Findings or a clean pass           |
| `refine`  | `check` reports structural or clarity issues      | Approve local hygiene edits and verify meaning is preserved    | Clearer artifact with same meaning |
| `tend`    | You know the intended refinement for one artifact | Explain the change, review impact, and approve edits           | A layer updated in place           |
| `weed`    | The real problem is divergence between artifacts  | Choose which side should move, then approve the correction     | Layers realigned and re-verified   |

## Workflow Cheat Sheet

Think of the sequence as a loop rather than a one-off command.

1. Start with `check` when you are unsure whether the layer is sound.
2. Use `refine` when `check` reports structural or clarity issues and intended meaning should remain stable.
3. Use `tend` when the change is local to one layer and the intent is already agreed.
4. Use `weed` when the discrepancy spans requirements, vocabulary, architecture, specs, or implementation and requires a human decision.
5. Finish with `check` again if you want a final validation pass after convergence.

For a change at a durable layer, `weed` each layer below it in turn (a vocabulary change runs `/gybis-vocab-weed`, then `/gybis-arch-weed`, then `/gybis-spec-weed`). If the change reaches specifications that already have code and tests, run `/gybis-spec-propagate` before `/gybis-spec-weed`. Repeat the loop until every affected layer passes `check` and a fresh pass produces no further change.

## Use Cases

### Find the Right Command

Use this when you know what kind of work you need to do, but not which gybis command to run first.

1. Run `/gybis-help` to see available commands, grouped by command family.
2. Choose the command family that matches the layer you need to work in before making changes further downstream.

Outcome: you start from the right command family instead of guessing from the full command list.

### Start a New Repository

Use this when you are building a new system and want durable constraints established before implementation.

1. Run `/gybis-req-elicit` to establish requirements with stakeholders via grilling interview rounds.
2. Run `/gybis-req-propagate` to derive initial `vocabulary.md` from the requirement terms, then validate it with `/gybis-vocab-check`.
3. Run `/gybis-vocab-propagate` to derive initial `architecture.md` from requirements and vocabulary, then validate it with `/gybis-arch-check`.
4. Run `/gybis-arch-propagate` to derive initial behavioral specifications, then validate them with `/gybis-spec-check`.
5. Run `/gybis-spec-propagate` to derive initial code and test scaffolding.

Outcome: the project starts from requirements, vocabulary, architecture, and specifications rather than implementation-first drift.

### Adopt gybis in an Existing Repository

Use this when code and tests already exist and you need to recover durable project truth from the current system.

1. Run `/gybis-spec-distill` to extract behavioral specifications from existing code and tests, then validate them with `/gybis-spec-check`.
2. Run `/gybis-arch-distill` to derive architecture from those specifications and the current implementation, then validate it with `/gybis-arch-check`.
3. Run `/gybis-vocab-distill` to extract vocabulary from architecture, specifications, and implementation, then validate it with `/gybis-vocab-check`.
4. Run `/gybis-req-distill` to distill the initial requirement set from vocabulary, architecture, specifications, and implementation, then validate it with `/gybis-req-check`.

Outcome: an existing codebase is brought under explicit requirements, vocabulary, architecture, and specification governance.

### Upgrade an Existing gybis Installation

Use this when updating this repository's existing gybis installation from a gybis repository. In this file, the **gybis command bundle** means the set of files in `gybis/.agents/skills/` that gets copied into this repository's `.agents/skills/` directory. The command bundle and this repository's project memory are separate concerns: update the former, then migrate the latter only when the migration command identifies a recognized legacy format.

#### Before You Start

- The command files come from the gybis repository: <https://github.com/TheCodeSojourner/gybis>. Use that same source for the whole upgrade.
- Finish or deliberately pause any current work before upgrading. If a gybis session is active, run `/gybis-fini` using the existing installation to save its session state before replacing `.agents/skills/`.
- Keep unrelated repository work separate from the upgrade. The upgrade diff should contain only changes to `.agents/skills/` and, if approved, selected files under `mementum/`.
- Make sure this repository's project-owned files are backed up or recoverable before upgrading: `mementum/state.md` (working memory), `mementum/index.md` (knowledge index), and locally edited `GYBIS-README.md` / `GYBIS-DEV-WORKFLOW.md`. These are durable project content.
- Do not run the full installation copy against this existing installation. It can overwrite this repository's live `mementum/` directory and other project artifacts.

#### Update the Command Bundle

From this repository, copy only the bundled skills:

```bash
rm -rf .agents/skills/gybis-* .agents/skills/internal && cp -ra <pathToGybisDirectory>/gybis/.agents/skills/. .agents/skills/
```

This updates command implementations, internal Allium adapters, and runtime compatibility checks. Remove first, because `cp` merges rather than replaces: a plain copy would leave skills the bundle has removed or renamed behind. That `rm` removes only gybis-owned entries (`gybis-*` skill directories and `internal/`), and `&&` copies only if the removal succeeds. Skills from other tools in `.agents/skills/` are preserved untouched and are neither removed nor overwritten. It does not update specifications, source code, tests, or Mementum data. Review the resulting skill-file diff, especially `internal/gybis-allium-runtime-check`, `internal/gybis-internal-skill-check`, and `internal/gybis-ref-check`.

The command-bundle copy installs skills only; it does not create the top-level stage directories. Their absence is normal for a repository that predates a layer, and most are created on demand:

- `requirements/` — created by the first `/gybis-req-elicit` or `/gybis-req-distill`. Until then, other `/gybis-req-*` commands report `requirements/ not found`.
- `specs/` — created by `/gybis-arch-propagate` or `/gybis-spec-distill`. Specification commands treat an absent or empty directory as `NO_SPECS`, an absence-of-work result rather than a failure.
- `mementum/` — created by no command. If it is absent, seed the bundle's empty OKF store before starting the session (`cp` creates the directory):

```bash
cp -ra <pathToGybisDirectory>/gybis/mementum/. mementum/
```

Verify the external prerequisite before running specification commands:

```bash
allium --version  # current bundle requirement: 3.5.3 or newer
```

#### Inspect and Migrate Mementum

Start a session with `/gybis-init` using the new installation. This loads the Nucleus and Mementum operating context and completes the session startup gate.

From that initialized session, run `/gybis-memory-migrate`:

```text
/gybis-memory-migrate
```

Migration and initialization are separate operations. Migration inspects `mementum/`, classifies the store, and shows an exact preview before it writes anything. Handle the result as follows:

| Result                    | Meaning                                                                  | Next step                                                                         |
| ------------------------- | ------------------------------------------------------------------------ | --------------------------------------------------------------------------------- |
| `NO_MIGRATION_REQUIRED`   | The store already conforms to OKF v0.1.                                  | Continue without migration.                                                       |
| `MIGRATABLE_LEGACY`       | Recognized legacy files can be converted deterministically.              | Review the file list and before/after preview, then explicitly approve or cancel. |
| `INITIALIZATION_REQUIRED` | No Mementum store exists to migrate.                                     | Seed the store as described above, then rerun migration.                          |
| `AMBIGUOUS`               | A file is malformed, unknown, conflicting, or otherwise unsafe to infer. | Resolve the reported ambiguity manually, then rerun the command.                  |

If you approve a migration, it must preserve memory and knowledge bodies verbatim, leave conformant files unchanged, and leave `mementum/state.md` unchanged. Continue only after the command reports `MIGRATION_VALIDATED`. It halts without changes for malformed or ambiguous data.

#### Validate Specifications

If the target contains `.allium` specifications, run the relevant scope:

```text
/gybis-spec-check
```

When no `.allium` files exist, `NO_SPECS` is expected. It is an absence-of-work result, not an Allium compatibility failure; the bundle upgrade can finish, and specifications can be created later. For a target that does contain specifications, this pass smoke-tests the upgraded Allium adapters and runtime gate against the installed CLI; the version preflight only confirms the executable is supported, not that its JSON contract matches.

#### Commit the Upgrade

Review the final diff, then commit the command-bundle update and any approved Mementum migration as a separately reviewable change. Keep the bundle update and migration history visible; do not silently fold either into unrelated project work.

Outcome: the target adopts the newer command bundle and, when required, OKF-compatible Mementum storage without losing or silently rewriting its durable memory.

### Explain System Intent to Different Audiences

Use this when onboarding developers, briefing stakeholders, or turning project truth into audience-specific explanations.

1. Run `/gybis-req-describe`, `/gybis-vocab-describe`, `/gybis-arch-describe`, and `/gybis-spec-describe {concern|domain|all}` for non-technical or business-facing explanations.
2. Run `/gybis-req-explain`, `/gybis-vocab-explain`, `/gybis-arch-explain`, and `/gybis-spec-explain {concern|domain|all}` for developer-facing explanations.

Outcome: the same requirements, vocabulary, architecture, and specifications can be communicated clearly to both technical and non-technical readers.

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

1. Run `/gybis-fini` to persist session state before termination.
2. Start the next session with `/gybis-init` so the recorded state is brought back into working context.

Outcome: each session leaves behind durable memory instead of losing project knowledge at the chat boundary.

---

## Architecture Principles

- **Organize by durability** — Structure things by how long they will likely last.
- **Hierarchy of abstractions:** why > what > how.
- **Layered system:** requirements > vocabulary > S5 > S4 > S3 > S2 > S1 > specs > tests > code — stricter, more durable layers constrain looser, more transient ones.
  Requirements, vocabulary, and S5..S1 are durable constraint layers. Together they constrain specifications, which then constrain tests, which then constrain code.
- **No flat structures.** Everything has its place in the hierarchy.
- **Top-down only, with no bypassing.** Higher layers constrain lower layers. Requirements > Vocabulary > Architecture (S5 ... S1) > Specs > Tests > Code. Lower layers never constrain higher layers.
- **Violations halt.** Skipping or reversing a layer surfaces immediately and stops progress rather than proceeding silently.
- **Drift surfaces automatically.** When code, tests, or behavior diverge from the durable constraints (requirements, vocabulary, architecture, specification), drift is detected, surfaced, and halted.

### Requirements Layer

Requirements are the top layer of the stack and the durable record of stakeholder intent.

- **Canonical form:** Dependency-ordered module files under `requirements/` hold `REQ-<DOMAIN>-NNN` clauses in nucleus lambda notation, rendered for humans on demand by describe/explain.
- **Constraint hierarchy:** Requirements constrain vocabulary, vocabulary constrains architecture, architecture constrains specifications, specifications constrain tests, and tests constrain code.
- **Durability:** Requirements are the most durable layer. They are elicited from stakeholders with `/gybis-req-elicit` and amended with `/gybis-req-tend` before any downstream layer changes.

### Vocabulary Layer

Vocabulary is the durable, human-agreed domain language that constrains downstream architecture and specifications.

- **Ubiquitous language:** `vocabulary.md` captures canonical terms, definitions, and relationships.
- **Constraint hierarchy:** Requirements constrain vocabulary, vocabulary constrains architecture, architecture constrains specifications, specifications constrain tests, and tests constrain code.
- **Durability:** Vocabulary is more durable than architecture. It is bootstrapped from requirements for new systems or distilled before architecture for existing systems.

---

## Architecture Philosophy

Requirements, vocabulary, and architecture describe system-level constraints that drive behavior specifications that drive tests, that drive code.

- **Constrain top-down.** Architecture governs specification, tests, and code, not the reverse.
- **Governance flow:** Requirements, vocabulary, architecture, specification, tests, and implementation must remain aligned.

---

## Specification Philosophy

Specifications describe code **behavior**, not implementation.

- **/gybis-arch-weed** checks for divergence between architecture and specs, then surfaces explicit resolution options with human approval.
- **/gybis-spec-distill** extracts behavioral specifications from existing code, then surfaces them for review and refinement.
- **Do not implement before specifying.** Implementation details are replaceable; specifications are the system's source of truth.
- **Workflow:** propagate or distill → check → refine → tend → weed.
- **Governance flows:** AI is governed by formalized behavior; behavior is governed by architecture.
- **Architecture alignment:** Only `/gybis-spec-propagate` and `/gybis-spec-weed` 
  apply architectural preferences; other spec skills are architecture-agnostic.

---

## Code as Replaceable Detail

- **Code and tests are replaceable.** They are ephemeral and interchangeable: code is the changing how, while the durable layers define the what and why.
- **Requirements, vocabulary, architecture, and specifications are project truth.** They define the system's expected behavior and constraints.
- **Never test before specifying.**
- **Never implement before testing.**

---

## Memory System

Memory tracks the state of six domains: **requirements, vocabulary, architecture, specification, tests, and code**.

- **Session persistence:** Session `n+1` is proportional to the sum of all prior encodings from sessions `1..n`.
- **No knowledge loss** across sessions. Decisions captured in one session are available to future sessions through memory recall commands.
- **Commands:** `/gybis-init` (session start), `/gybis-fini` (session end), `/gybis-memory-*` — persist state or restore from it.

---

### Session Memory Workflow

Each work session follows a structured lifecycle:

1. **Start:** `/gybis-init` — Orient → Recall → Ready
2. **End:** `/gybis-fini` — Persist memory → Terminate

Every session guarantees **no knowledge loss**.

---

## Authority Model

This section defines who decides what and when, for all non-memory operations.

- **Human commands come first.** The AI never takes initiative.
- **Human is the approval gate.** Every write operation requires human approval.
- **No autonomous AI actions, and no covert operations.** Every change is human-visible.
- **Invocation authorizes the command's writes.** Running a command is the human approval for the writes that command is defined to make. `distill` and `propagate` persist their artifacts autonomously once invoked; their output is reviewed afterwards through `check` and `weed`. Only writes nobody asked for are prohibited.

---

## Commands

| Skill Name                                                        | Description                                             |
| ----------------------------------------------------------------- | ------------------------------------------------------- |
| `/gybis-arch-check` (`/ga-check`)                                 | Validate architecture integrity & coherence             |
| `/gybis-arch-describe` (`/ga-describe`)                           | Describe arch in stakeholder prose or markdown          |
| `/gybis-arch-distill` (`/ga-distill`)                             | Create initial arch from specs                          |
| `/gybis-arch-explain` (`/ga-explain`)                             | Explain arch in dev prose or markdown                   |
| `/gybis-arch-propagate` (`/ga-propagate`)                         | Create initial specs from arch                          |
| `/gybis-arch-refine` (`/ga-refine`)                               | Refine architecture structure & clarity                 |
| `/gybis-arch-tend` (`/ga-tend`)                                   | Update arch with impact analysis                        |
| `/gybis-arch-weed` (`/ga-weed`)                                   | Resolve divergence between arch and specs               |
| `/gybis-fini`                                                     | Persist memory before terminate                         |
| `/gybis-help`                                                     | Show available commands                                 |
| `/gybis-init`                                                     | Initialize gybis AI context                             |
| `/gybis-memory-migrate` (`/gm-migrate`)                           | Migrate Mementum store to current format                |
| `/gybis-memory-orient` (`/gm-orient`)                             | Restore prev AI context                                 |
| `/gybis-memory-recall {topic}` (`/gm-recall {topic}`)             | Recall topic/summarize-latest                           |
| `/gybis-memory-store {insight}` (`/gm-store {insight}`)           | Store insight, or prompt for one                        |
| `/gybis-memory-synthesize` (`/gm-synthesize`)                     | Synthesize knowledge                                    |
| `/gybis-req-check` (`/gr-check`)                                  | Validate individual requirements, ordering, & coverage  |
| `/gybis-req-describe` (`/gr-describe`)                            | Describe requirements in stakeholder prose or markdown  |
| `/gybis-req-distill` (`/gr-distill`)                              | Create initial requirements from vocab/arch/specs/code  |
| `/gybis-req-elicit` (`/gr-elicit`)                                | Elicit requirements via grilling interview rounds       |
| `/gybis-req-explain` (`/gr-explain`)                              | Explain requirements in dev prose or markdown           |
| `/gybis-req-propagate` (`/gr-propagate`)                          | Annotate specs/tests with REQ traceability              |
| `/gybis-req-refine` (`/gr-refine`)                                | Refine requirements structure & clarity                 |
| `/gybis-req-tend` (`/gr-tend`)                                    | Update requirements with impact analysis                |
| `/gybis-req-weed` (`/gr-weed`)                                    | Resolve divergence between requirements and downstream  |
| `/gybis-spec-check` (`/gs-check {concern\|domain\|all}`)          | Validate and repair spec syntax                         |
| `/gybis-spec-describe` (`/gs-describe {concern\|domain\|all}`)    | Describe specs in stakeholder prose or markdown         |
| `/gybis-spec-distill` (`/gs-distill`)                             | Create initial specs from code/tests                    |
| `/gybis-spec-explain` (`/gs-explain {concern\|domain\|all}`)      | Explain specs in dev prose or markdown                  |
| `/gybis-spec-propagate` (`/gs-propagate {concern\|domain\|all}`)  | Create initial code/tests                               |
| `/gybis-spec-refine` (`/gs-refine`)                               | Refine specs structure & clarity                        |
| `/gybis-spec-tend` (`/gs-tend`)                                   | Update specs with impact analysis                       |
| `/gybis-spec-weed` (`/gs-weed`)                                   | Resolve divergence between specs and code               |
| `/gybis-vocab-check` (`/gv-check`)                                | Validate vocabulary.md syntax & semantics               |
| `/gybis-vocab-describe` (`/gv-describe`)                          | Describe vocabulary in stakeholder prose                |
| `/gybis-vocab-distill` (`/gv-distill`)                            | Extract vocabulary from arch/specs/code                 |
| `/gybis-vocab-explain` (`/gv-explain`)                            | Explain vocabulary in dev prose                         |
| `/gybis-vocab-propagate` (`/gv-propagate`)                        | Create initial architecture from req + vocab            |
| `/gybis-vocab-refine` (`/gv-refine`)                              | Refine vocabulary structure & clarity                   |
| `/gybis-vocab-tend` (`/gv-tend`)                                  | Update vocabulary with impact analysis                  |
| `/gybis-vocab-weed` (`/gv-weed`)                                  | Resolve divergence between vocab and downstream         |

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