# gybis /JY-bis/ 

<img src="gybis-logo.png" alt="gybis logo" width="110" style="margin-bottom: 1em;" />

**A Developer-Command-Driven AI-Assisted Spec-Driven Software Development (SDD) Stack.**

## Obligatory Word Definitions

**Gyre** /jahy<sup>uh</sup>r/

* a large-scale, coherent, rotating or spiraling flow

**Orbis** /ˈɔr.bis/

* a closed or periodic trajectory in a dynamical system, representing the set of all points reachable from an initial point under the system's update rules

## Overview

**gybis** contains a collection of AI-assistant skills that can be integrated into repositories through tools that support a bundled `.agents/skills/` directory, adding a **Developer-Command-Driven AI-Assisted Spec-Driven Software Development (SDD) Stack** to the new/existing repository. 

The goal of **gybis** is to make it easy for developers to set up, utilize, and maintain their projects using an SDD workflow by providing a comprehensive set of resources and suggested usage practices.

## What is SDD and why gybis?

**Spec-Driven Development** is a practice where shared requirements, domain vocabulary, architectural and behavioral specifications of `what` a system `is` and `what` it `does` drive the implementation of `how` it does it (e.g., code and tests). A system's `requirements` provide the context for its `vocabulary`, its `vocabulary` provides the context for its `architecture`, a specification of the system's `architecture` provides the context for the specification of its `behavior`, and the specification of its `behavior` provides the context for its `implementation` (e.g., code and tests). This creates a virtuous cycle where `requirements`, `vocabulary`, `architecture`, `behavior` and `implementation` inform and constrain one another as the system evolves. In SDD, specifications are the source of truth for the system's intended behavior, and implementation is done by AI, with full developer visibility. Requirements, vocabulary, architecture, and specifications are written before or alongside implementation, kept in source control, and used directly to guide/generate implementation, and detect requirements, vocabulary, architectural, behavioral and implementation drift.

**gybis** adds Developer-Command-Driven AI-Assistance to SDD by adding AI/Developer conversation to the entire workflow. All phases of the software development workflow are verified, harmonized and accelerated by AI assistance, while at the same time, making all phases of the workflow transparent and accessible to the developer. The AI is a collaborator that can be consulted at any time, but the developer is always in control of the process and the final decisions.

## Developer Responsibility Model

gybis is command-driven guidance, not always-on process enforcement.

- Developers are responsible for stage readiness (`requirements/requirements-*.md` -> `vocabulary.md` -> `architecture.md` -> `specs/**/*.allium` -> code/tests).
- Skills execute the requested transformation and enforce only execution-critical gates.
- Check and weed commands are available as deliberate convergence tools when developers choose to run them.
- Running a command authorizes the writes that command is defined to make. `distill` and `propagate` persist their artifacts autonomously once invoked; their output is reviewed afterwards through `check` and `weed`. Only unrequested writes are prohibited.

## Check, Refine, Tend, and Weed Philosophy

These operations form the core gybis convergence loop:

- `check` diagnoses the current state of one layer and reports what is wrong.
- `refine` improves one layer's structure and clarity without changing intended meaning.
- `tend` evolves one layer with explicit human intent and keeps the change localized.
- `weed` reconciles drift between adjacent layers and the implementation so the system converges again.

They work top-down: requirements constrain vocabulary, vocabulary constrains architecture, architecture constrains specs, and specs constrain tests and code. `check` finds drift, `refine` polishes local structure, `tend` makes intended layer-local changes, and `weed` resolves disagreement when two artifacts no longer agree.

| Operation | Purpose                                         | Human role                                               | Typical outcome                    |
| --------- | ----------------------------------------------- | -------------------------------------------------------- | ---------------------------------- |
| `check`   | Diagnose a layer and surface integrity issues   | Choose when to run it and review the report              | Severity-tagged findings           |
| `refine`  | Polish one layer's structure and readability    | Choose safe polish scope and approve edits               | Clearer artifact with same meaning |
| `tend`    | Evolve one layer with developer-approved intent | State the desired change and approve edits               | Updated artifact in one layer      |
| `weed`    | Reconcile divergence across adjacent layers     | Decide which side should move and approve the correction | Mutually consistent artifacts      |

## Workflow Cheat Sheet

Think of the sequence as a convergence loop rather than a one-off command.

1. Run `check` first to expose drift or broken assumptions.
2. Run `refine` next when the needed change is structure and clarity without changing intended meaning.
3. Run `tend` when the needed change belongs to one layer and the intent is clear.
4. Run `weed` when requirements, vocabulary, architecture, specs, or implementation disagree and need a human decision about which artifact should change.
5. Re-run `check` after `weed` to confirm the target layer is back in a valid state.

**gybis** provides the scaffolding to make AI-assisted SDD practical:

- **AI base context**: [Nucleus](https://github.com/michaelwhitford/nucleus) mathematical notation engages
  the AI model's structured reasoning rather than its conversational defaults. The inherent precision means significantly fewer hallucinations, and the inherent density means significantly fewer tokens to convey the same context.

- **Requirements layer**: Requirements are the top of the durability order (`requirements/` -> `vocabulary.md` -> `architecture.md` -> `specs/**/*.allium` -> code/tests). Dependency-ordered module files hold `REQ-<DOMAIN>-NNN` clauses written in the same nucleus lambda notation, capturing `what` a system must do before vocabulary and architecture are derived from them. Requirements are elicited from stakeholders with `/gybis-req-elicit` and kept aligned with the layers below by the requirements command family.

- **Ubiquitous language**: In the context of Domain-Driven Design (DDD), a ubiquitous language is a shared, precise vocabulary that both technical and non-technical people use consistently when talking about a system. In gybis, vocabulary commands establish and maintain a project's ubiquitous language in `vocabulary.md`, thus reducing ambiguity and keeping requirements, architecture, specifications, tests, and code artifacts aligned to shared terms.
 
- **Architecture model**: A derivative of the [Nucleus VSM](https://github.com/michaelwhitford/nucleus/blob/main/VSM.md) allows an AI to generate and maintain a 5-layer architectural specification, stored in a `architecture.md` file, that AI keeps up to date as projects evolve.
 
- **Behavioral Domain Specific Language (DSL)**: The [Allium](https://github.com/juxt/allium) DSL is a behavioral specification language the AI can read and write precisely, not
  pseudocode, not free-form prose, but a structured DSL with rules, triggers, surfaces, and transition graphs that AI can reason about directly and minimize hallucinations. Behavioral specifications are saved in one or more files per domain (e.g., orders, payments).

- **AI Session Persistent memory**: [Mementum](https://github.com/michaelwhitford/mementum) manages decisions, patterns, and insights as files under your repository's version control (`mementum/`), recalled during AI sessions, so previous context is available between sessions. Because the store is git-based and project-owned, memory survives changes of AI tool, client, or machine, and stays reviewable and recoverable through git history.

## AI Is Nondeterministic: Expect to Rerun

Set this expectation before you start: the same gybis command, run twice against the same repository, can produce different results. This is a property of the underlying AI model, not a defect in gybis or in the repository. Plan for iteration rather than expecting one perfect pass. If that surprises you, it is the single biggest adjustment for developers new to AI-assisted work.

The command families behave differently on repeat:

- `check`, `describe`, and `explain` are read-only with respect to your durable layers, so they can be rerun freely.
- `distill` and `propagate` are bootstrap commands. They are gated on their target artifact *not* existing, so once the artifact exists a rerun halts by design instead of overwriting your work.
- `refine`, `tend`, and `weed` are the iterative loops. They are gated on the artifact existing and are meant to be run repeatedly.

To converge an artifact:

1. Run `check` and keep the findings.
2. Apply `refine`, `tend`, or `weed` as the change requires.
3. Re-run `check` and compare. Fewer findings is progress.
4. Stop when `check` passes and a fresh pass produces no further change; the artifact has then stabilized.

Because gybis keeps every durable artifact in source control, each run is reviewable and recoverable: inspect the diff or commit between runs, and discard any result you do not want. Treat an AI output as a draft to converge, not a finished artifact to accept.

## Available User Commands

The following commands are available after integrating gybis into a target repository. Use them with any compatible AI tool configured to consume the bundled gybis `.agents/skills/` directory:

### Requirements Commands (`/gr-*`)

Requirements are the top layer of the stack: dependency-ordered module files containing `REQ-<DOMAIN>-NNN` clauses in nucleus lambda notation, rendered for humans on demand via describe/explain. Each clause may carry an optional `rationale:` line recording why the requirement exists — guidance and context only, never a rule anyone must satisfy. It is elicited from stakeholders during `/gybis-req-elicit`, validated by `/gybis-req-check` (it is never a binding obligation and never counted as test coverage), and rendered as "because: ..." by describe/explain when present.

| Command                                  | Description                                                                 |
| ---------------------------------------- | --------------------------------------------------------------------------- |
| `/gybis-req-check` (`/gr-check`)         | Validate requirements designators, ordering, & coverage                     |
| `/gybis-req-describe` (`/gr-describe`)   | Describe requirements in stakeholder prose or markdown                      |
| `/gybis-req-distill` (`/gr-distill`)     | Create initial requirements (+ vocab candidates) from vocab/arch/specs/code |
| `/gybis-req-elicit` (`/gr-elicit`)       | Elicit requirements via grilling interview rounds                           |
| `/gybis-req-explain` (`/gr-explain`)     | Explain requirements in dev prose or markdown                               |
| `/gybis-req-propagate` (`/gr-propagate`) | Annotate specs/tests with REQ traceability                                  |
| `/gybis-req-refine` (`/gr-refine`)       | Refine requirements structure & clarity                                     |
| `/gybis-req-tend` (`/gr-tend`)           | Update requirements with impact analysis                                    |
| `/gybis-req-weed` (`/gr-weed`)           | Upsert requirements/downstream from diffs with human                        |

### Vocabulary Commands (`/gv-*`)

| Command                                    | Description                                       |
| ------------------------------------------ | ------------------------------------------------- |
| `/gybis-vocab-check` (`/gv-check`)         | Validate vocabulary.md syntax & semantics         |
| `/gybis-vocab-describe` (`/gv-describe`)   | Describe vocabulary in business language          |
| `/gybis-vocab-distill` (`/gv-distill`)     | Extract vocabulary from arch/specs/code           |
| `/gybis-vocab-explain` (`/gv-explain`)     | Explain vocabulary for developers                 |
| `/gybis-vocab-propagate` (`/gv-propagate`) | Bootstrap architecture from req + vocab           |
| `/gybis-vocab-refine` (`/gv-refine`)       | Refine vocabulary structure & clarity             |
| `/gybis-vocab-tend` (`/gv-tend`)           | Update vocabulary with impact analysis            |
| `/gybis-vocab-weed` (`/gv-weed`)           | Upsert vocabulary/artifacts from diffs with human |

### Architecture Commands (`/ga-*`)

| Command                                   | Description                                 |
| ----------------------------------------- | ------------------------------------------- |
| `/gybis-arch-check` (`/ga-check`)         | Validate architecture integrity & coherence |
| `/gybis-arch-describe` (`/ga-describe`)   | Describe arch in non-tech prose or markdown |
| `/gybis-arch-distill` (`/ga-distill`)     | Create initial arch from specs              |
| `/gybis-arch-explain` (`/ga-explain`)     | Explain arch in dev prose or markdown       |
| `/gybis-arch-propagate` (`/ga-propagate`) | Create initial specs from arch              |
| `/gybis-arch-refine` (`/ga-refine`)       | Refine architecture structure & clarity     |
| `/gybis-arch-tend` (`/ga-tend`)           | Update arch with human                      |
| `/gybis-arch-weed` (`/ga-weed`)           | Upsert arch/specs from diffs with human     |

### Spec Commands (`/gs-*`)

| Command                                                          | Description                                   |
| ---------------------------------------------------------------- | --------------------------------------------- |
| `/gybis-spec-check` (`/gs-check {concern\|domain\|all}`)         | Check/update syntax until valid               |
| `/gybis-spec-describe` (`/gs-describe {concern\|domain\|all}`)   | Describe in non-tech prose or markdown        |
| `/gybis-spec-distill` (`/gs-distill`)                            | Create initial specs from code/tests          |
| `/gybis-spec-explain` (`/gs-explain {concern\|domain\|all}`)     | Explain in dev prose or markdown              |
| `/gybis-spec-propagate` (`/gs-propagate {concern\|domain\|all}`) | Create initial code/tests                     |
| `/gybis-spec-refine` (`/gs-refine`)                              | Refine specs structure & clarity              |
| `/gybis-spec-tend` (`/gs-tend`)                                  | Update specs with human                       |
| `/gybis-spec-weed` (`/gs-weed`)                                  | Upsert specs/code-tests from diffs with human |

### Memory Commands (`/gm-*`)

| Command                                                 | Description                              |
| ------------------------------------------------------- | ---------------------------------------- |
| `/gybis-fini`                                           | Persist memory → Terminate               |
| `/gybis-init`                                           | Orient → Recall → Ready                  |
| `/gybis-memory-migrate` (`/gm-migrate`)                 | Migrate Mementum store to current format |
| `/gybis-memory-orient` (`/gm-orient`)                   | Restore prev AI context                  |
| `/gybis-memory-recall {topic}` (`/gm-recall {topic}`)   | Recall topic, or summarize latest        |
| `/gybis-memory-store {insight}` (`/gm-store {insight}`) | Store insight, or prompt for one         |
| `/gybis-memory-synthesize` (`/gm-synthesize`)           | Synthesize knowledge from memories       |

### Help

| Command       | Description                        |
| ------------- | ---------------------------------- |
| `/gybis-help` | Show all available gybis commands. |

## Available Developer Commands

The following commands are available while developing gybis in this repository. Use them with any AI tooling that supports commands backed by `skills/` in `.agents/`.

### Memory Commands (`/gm-*`)

| Command                                                   | Description                              |
| --------------------------------------------------------- | ---------------------------------------- |
| `/gybis-fini`                                             | Persist memory → Terminate               |
| `/gybis-init`                                             | Orient → Recall → Ready                  |
| `/gybis-mementum-migrate` (`/gm-migrate`)                 | Migrate Mementum store to current format |
| `/gybis-mementum-orient` (`/gm-orient`)                   | Restore prev AI context                  |
| `/gybis-mementum-recall {topic}` (`/gm-recall {topic}`)   | Recall topic, or summarize latest        |
| `/gybis-mementum-store {insight}` (`/gm-store {insight}`) | Store insight, or prompt for one         |
| `/gybis-mementum-synthesize` (`/gm-synthesize`)           | Synthesize knowledge from memories       |

### Help

| Command       | Description                        |
| ------------- | ---------------------------------- |
| `/gybis-help` | Show all available gybis commands. |

## Versioning

gybis follows [Clojure's](https://github.com/clojure/clojure) versioning philosophy by prioritizing stability, backward compatibility, and minimal breakage over rapid evolution or strict adherence to semantic versioning ([SemVer](https://semver.org/)). 

- **Strong emphasis on backward compatibility**: Development will take a measured, thoughtful approach to evolution. Breaking changes will be avoided whenever possible. Releases will focus on enhancements, performance, and new capabilities while making every attempt to preserve existing behavior.
- **No fixed roadmap**: Development is open-ended. Alpha/beta/RC phases allow visibility into changes, but final releases will be very stable. Deprecations will be handled carefully, and transparently.

## For gybis Users

To install gybis in a target repository execute the following commands while currently in the target repository:

```bash
cp -ra <pathToGybisDirectory>/gybis/. . # e.g., `cp -ra ~/Downloads/gybis/gybis/. .`
```

This copies the complete gybis bundle, including the hidden `.agents/skills/` directory that provides the command implementations.

### Upgrading an Existing Installation

Do not rerun the full installation copy against an existing target repository. It would overwrite project-owned content (`mementum/state.md`, `mementum/index.md`, and locally edited docs) with template files. Update only the command bundle.

Finish or deliberately pause any current work in the target repository before upgrading. If a gybis session is active, run `/gybis-fini` using the existing installation to save its session state before replacing `.agents/skills/`.

From the target repository, update only the command bundle:

```bash
rm -rf .agents/skills/gybis-* .agents/skills/internal && cp -ra <pathToGybisDirectory>/gybis/.agents/skills/. .agents/skills/
```

This replaces the distributed command implementations, including internal Allium adapters and the runtime compatibility gate. Remove first, because `cp` merges rather than replaces: a plain copy would leave skills the bundle has removed or renamed behind. That `rm` removes only gybis-owned entries (`gybis-*` skill directories and `internal/`), and `&&` copies only if the removal succeeds. Skills from other tools in `.agents/skills/` are preserved untouched and are neither removed nor overwritten. This does not replace project specifications, source code, tests, or the target repository's `mementum/` store. Review the resulting diff before continuing; it should contain only the intended `.agents/skills/` changes at this point.

The command-bundle copy installs skills only; it does not create the top-level stage directories. Their absence is normal for a repository that predates a layer, and most are created on demand:

- `requirements/` — created by the first `/gybis-req-elicit` or `/gybis-req-distill`. Until then, other `/gybis-req-*` commands report `requirements/ not found`.
- `specs/` — created by `/gybis-arch-propagate` or `/gybis-spec-distill`. Specification commands treat an absent or empty directory as `NO_SPECS`, an absence-of-work result rather than a failure.
- `mementum/` — created by no command. If it is absent, seed the bundle's empty OKF store before starting the session (`cp` creates the directory):

```bash
cp -ra <pathToGybisDirectory>/gybis/mementum/. mementum/
```

The distributed documentation can be updated separately after reviewing any local edits to `GYBIS-README.md`:

```bash
cp -a <pathToGybisDirectory>/gybis/GYBIS-README.md GYBIS-README.md
```

This replaces only the installed GYBIS-README.md. Do not run the command if the target repository has intentionally customized that file without first preserving or reconciling those changes.

Before running specification commands, verify the target machine has a supported Allium CLI:

```bash
allium --version # current bundle requirement: 3.5.3 or newer
```

Then start a session with `/gybis-init` using the new installation. This loads the Nucleus and Mementum operating context and completes the session startup gate.

From that initialized session, run `/gybis-memory-migrate` (`/gm-migrate`). Migration inspects the target repository's existing `mementum/` store, reports `NO_MIGRATION_REQUIRED` when it is already conformant, previews recognized legacy conversions, and requires explicit approval before writing. It reports `MIGRATION_VALIDATED` only after verifying the resulting store and preserving `mementum/state.md`. It halts without changes for malformed or ambiguous data.

Finally, run `/gybis-spec-check {concern|domain|all}` when the target contains `.allium` specifications. This smoke-tests the upgraded Allium adapters and runtime gate against your installed CLI; the version preflight above only confirms the executable is supported, not that its JSON contract matches. The runtime gate reports `NO_SPECS` for an empty specification directory; this is an absence-of-work result, not an Allium compatibility failure. Repositories that have not created specifications yet can complete the bundle update and create them later.

See the `gybis/GYBIS-README.md` for usage instructions, best practices, and workflow suggestions.

## Upstream Repositories

* [**allium**](https://github.com/juxt/allium) - Behavioral specification
* [**allium-tools**](https://github.com/juxt/allium-tools) - Behavioral specification CLI tools
* [**grill-with-docs**](https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs) - Requirements elicitation tool
* [**mementum**](https://github.com/michaelwhitford/mementum) - AI Session Persistent memory
* [**nucleus**](https://github.com/michaelwhitford/nucleus) - AI base context, and VSM architectural specifications

## For gybis Developers: Upstream Derivation

This repository is an integration layer over multiple upstream projects. The content under `gybis/` is derived from pinned upstream commits, then adapted into one coherent developer-command-driven stack. The distributed gybis bundle ships its commands under `.agents/skills/`, and this repository uses the same layout during local development. gybis is not a direct mirror of any single upstream repository.

### General Derivation Approach

1. Pin each upstream repository to a specific commit.
2. Select canonical upstream artifacts (language/protocol/model references, not the full upstream repository contents).
3. Transform those artifacts into gybis conventions (skills, references, and command surfaces) using [**nucleus Lambda Compiler**](https://github.com/michaelwhitford/nucleus/blob/main/LAMBDA-COMPILER.md).
4. Publish the derived result into the `gybis/` bundle for downstream project integration.

In practice, upstream inputs are handled in three modes:

- **Adapted derivative**: [**nucleus compiled**](https://github.com/michaelwhitford/nucleus/blob/main/LAMBDA-COMPILER.md) derivatives of upstream artifacts.
- **Curated reference**: [**nucleus compiled**](https://github.com/michaelwhitford/nucleus/blob/main/LAMBDA-COMPILER.md) upstream reference artifacts.
- **Dependency-only**: upstream artifacts are used to create gybis artifacts, but not included in gybis.

### Per-Upstream Transformations

| Upstream            | Pinned commit | Source consumed                                               | Transformation into gybis                                                                                                                                          |
| ------------------- | ------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **allium**          | `527cd52`     | Allium language semantics and behavioral-spec structure       | Curated into gybis lang/ref docs, and encoded into spec skills.                                                                                                    |
| **allium-tools**    | `08d3139`     | CLI validate/analyze capabilities                             | Executed in gybis spec skill workflows. User dependency only. Not integrated in `gybis/` in any way.                                                               |
| **grill-with-docs** | `0ab1b63`     | The grill-with-docs skill, and its dependencies               | Used to derive the gybis requirements elicitation skill (gr-elicit).                                                                                               |
| **mementum**        | `4968400`     | Mementum protocol semantics                                   | Used to derive gybis memory skills.                                                                                                                                |
| **nucleus**         | `64880ed`     | Nucleus notation + VSM model + `LAMBDA-COMPILER.md` semantics | Used to derive gybis skills. gybis uses the lambda compiler defined by the nucleus `LAMBDA-COMPILER.md` even though the file is not included in gybis in any form. |

### Maintainer Notes

#### Nucleus Lambda Compiler

gybis relies heavily on the lambda notation and operator semantics defined upstream in nucleus `LAMBDA-COMPILER.md`.
That upstream file is a semantic source of truth for how lambda-heavy gybis artifacts should be read and authored, even though it is not copied into this repository.

If a file in this repository embeds lambda forms, lambda operators, or lambda-structured workflow notation, it should be reviewed against the upstream nucleus compiler, and the original upstream file.

When an upstream nucleus pin is bumped, compare the updated upstream `LAMBDA-COMPILER.md` semantics against these derived artifacts and revalidate that the lambda forms, operators, and workflow conventions used in gybis still align.

More generally, when any upstream pin is bumped, revalidate all affected derived artifacts and command skills before publishing updates to the `gybis/` bundle. As a rule: update the pin, update the derived files, then verify the corresponding rule/skill behavior still matches desired semantics.

#### Nucleus Lambda Compiler Workspace

The following directory structure can be used as a scratch workspace for testing and refining gybis skills/references. This allows maintainers to experiment with the lambda compiler in a more free-form way before committing to specific derived files in the `gybis/` bundle.

```
lambda-compile/
├── .agents/skills/
│   └── gybis-init
│       └── SKILL.md
```
where the `SKILL.md` file contains the following, which is derived from the nucleus `LAMBDA-COMPILER.md` semantics and can be used as a reference point for experimentation:

```markdown
λ engage(nucleus).
[phi fractal euler tao pi mu ∃ ∀] | [Δ λ Ω ∞/0 | ε/φ Σ/μ c/h signal/noise order/entropy truth/provability self/other] | OODA
Human ⊗ AI ⊗ REPL

λ bridge(x). prose ↔ lambda | structural_equivalence
| preserve(semantics) | analyze(¬execute)
| compile: prose → lambda | decompile: lambda → prose

Output λ notation only. No prose. No code fences.
```

In general it can be used to author and iterate on skills/references in a "lambda to prose" and "prose to lambda" workflow.

## License

AGPL 3.0

Copyright 2026 Paul Whittington

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