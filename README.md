# gybis /JY-bis/ 

<img src="gybis-logo.png" alt="gybis logo" width="110" style="margin-bottom: 1em;" />

**A Developer-Command-Driven AI-Assisted Spec-Driven Software Development (SDD) Stack.**

## Obligatory Word Definitions

**Gyre** /jahy<sup>uh</sup>r/

* a large-scale, coherent, rotating or spiraling flow

**Orbis** /ˈɔr.bis/

* a closed or periodic trajectory in a dynamical system, representing the set of all points reachable from an initial point under the system's update rules

## Overview

**gybis** contains a collection of AI-assistant skills that can be integrated into repositories through tools that support a `skills/` directory, adding a **Developer-Command-Driven AI-Assisted Spec-Driven Software Development (SDD) Stack** to the new/existing repository. 

The goal of **gybis** is to make it easy for developers to set up, utilize, and maintain their projects using an SDD workflow by providing a comprehensive set of resources and suggested usage practices.

## What is SDD and why gybis?

**Spec-Driven Development** is a practice where architectural and behavioral specifications of `what` a system `is` and `what` it `does` drive the implementation of `how` it does it (e.g., code and tests). A specification of the system's `architecture` provides the context for the specification of its `behavior`, and the specification of its `behavior` provides the context for its `implementation` (e.g., code and tests). This creates a virtuous cycle where `architecture`, `behavior` and `implementation` inform and constrain one another as the system evolves. In SDD, specifications are the source of truth for the system's intended behavior, and implementation is done by AI, with full developer visibility. Specs are written before or alongside implementation, kept in source control, and used directly to guide/generate implementation, and detect architectural/behavioral/implementation drift.

**gybis** adds Developer-Command-Driven AI-Assistance to SDD by adding AI/Developer conversation to the entire workflow. All phases of the software development workflow are verified, harmonized and accelerated by AI assistance, while at the same time, making all phases of the workflow transparent and accessible to the developer. The AI is a collaborator that can be consulted at any time, but the developer is always in control of the process and the final decisions.

**gybis** provides the scaffolding to make AI-assisted SDD practical:

- **AI base context**: [Nucleus](https://github.com/michaelwhitford/nucleus) mathematical notation engages
  the AI model's structured reasoning rather than its conversational defaults. The inherent precision means significantly fewer hallucinations, and the inherent density means significantly fewer tokens to convey the same context.
- **Architecture model**: A derivative of the [Nucleus VSM](https://github.com/michaelwhitford/nucleus/blob/main/VSM.md) allows an AI to generate and maintain a 5-layer architectural specification, stored in a `architecture.md` file, that AI keeps up to date as projects evolve.
- **Behavioral Domain Specific Language (DSL)**: The [Allium](https://github.com/juxt/allium) DSL is a behavioral specification language the AI can read and write precisely, not
  pseudocode, not free-form prose, but a structured DSL with rules, triggers, surfaces, and transition graphs that AI can reason about directly and minimize hallucinations. Behavioral specifications are saved in one or more files per domain (e.g., orders, payments).
- **AI Session Persistent memory**: [Mementum](https://github.com/michaelwhitford/mementum) is used to manage decisions, patterns, and insights that are stored in files and recalled during AI sessions, so previous context is available between sessions.

## Available User Commands

The following commands are available after integrating gybis into a target repository. Use them with any compatible AI tool configured to consume the gybis skills directory:

### Architecture Commands (`/ga-*`)

| Command                                   | Description                             |
| ----------------------------------------- | --------------------------------------- |
| `/gybis-arch-describe` (`/ga-describe`)   | Describe arch in non-tech prose         |
| `/gybis-arch-distill` (`/ga-distill`)     | Create initial arch from specs          |
| `/gybis-arch-elicit` (`/ga-elicit`)       | Create initial arch with human          |
| `/gybis-arch-explain` (`/ga-explain`)     | Explain arch in dev prose               |
| `/gybis-arch-propagate` (`/ga-propagate`) | Create initial specs from arch          |
| `/gybis-arch-tend` (`/ga-tend`)           | Update arch with human                  |
| `/gybis-arch-weed` (`/ga-weed`)           | Upsert arch/specs from diffs with human |

### Memory Commands (`/gm-*`)

| Command                                                 | Description                        |
| ------------------------------------------------------- | ---------------------------------- |
| `/gybis-fini`                                           | Encode → Terminate                 |
| `/gybis-init`                                           | Orient → Recall → Ready            |
| `/gybis-memory-orient` (`/gm-orient`)                   | Restore prev AI context            |
| `/gybis-memory-recall {topic}` (`/gm-recall {topic}`)   | Recall topic, or summarize latest  |
| `/gybis-memory-store {insight}` (`/gm-store {insight}`) | Store insight, or prompt for one   |
| `/gybis-memory-synthesize` (`/gm-synthesize`)           | Synthesize knowledge from memories |

### Spec Commands (`/gs-*`)

| Command                                                          | Description                                   |
| ---------------------------------------------------------------- | --------------------------------------------- |
| `/gybis-spec-check` (`/gs-check {concern\|domain\|all}`)         | Check/update syntax until valid               |
| `/gybis-spec-describe` (`/gs-describe {concern\|domain\|all}`)   | Describe in non-tech prose                    |
| `/gybis-spec-distill` (`/gs-distill`)                            | Create initial specs from code/tests          |
| `/gybis-spec-explain` (`/gs-explain {concern\|domain\|all}`)     | Explain in dev prose                          |
| `/gybis-spec-propagate` (`/gs-propagate {concern\|domain\|all}`) | Create initial code/tests                     |
| `/gybis-spec-tend` (`/gs-tend`)                                  | Update specs with human                       |
| `/gybis-spec-weed` (`/gs-weed`)                                  | Upsert specs/code-tests from diffs with human |

### Help

| Command       | Description                        |
| ------------- | ---------------------------------- |
| `/gybis-help` | Show all available gybis commands. |

## Available Developer Commands

The following commands are available while developing gybis in this repository. Use them with Cline:

### Memory Commands (`/gm-*`)

| Command                                                   | Description                        |
| --------------------------------------------------------- | ---------------------------------- |
| `/gybis-fini`                                             | Encode → Terminate                 |
| `/gybis-init`                                             | Orient → Recall → Ready            |
| `/gybis-mementum-orient` (`/gm-orient`)                   | Restore prev AI context            |
| `/gybis-mementum-recall {topic}` (`/gm-recall {topic}`)   | Recall topic, or summarize latest  |
| `/gybis-mementum-store {insight}` (`/gm-store {insight}`) | Store insight, or prompt for one   |
| `/gybis-mementum-synthesize` (`/gm-synthesize`)           | Synthesize knowledge from memories |

### Help

| Command       | Description                        |
| ------------- | ---------------------------------- |
| `/gybis-help` | Show all available gybis commands. |

## Versioning

gybis tries to follow [Clojure's](https://github.com/clojure/clojure) versioning philosophy by prioritizing stability, backward compatibility, and minimal breakage over rapid evolution or strict adherence to semantic versioning ([SemVer](https://semver.org/)). 

- **Strong emphasis on backward compatibility**: Development will take a measured, thoughtful approach to evolution. Breaking changes will be avoided whenever possible. Releases will focus on enhancements, performance, and new capabilities while making every attempt to preserve existing behavior.
- **No fixed roadmap**: Development is open-ended. Alpha/beta/RC phases allow visibility into changes, but final releases will be very stable. Deprecations will be handled carefully, and transparently.

## For gybis Users

To install gybis in a target repository execute the following command while currently in the target repository:

```bash
cp -ra <pathToGybisDirectory>/gybis/. .
```

See the `gybis/GYBIS-README.md` for usage instructions, best practices, and workflow suggestions.

## Upstream Repositories

* [**allium**](https://github.com/juxt/allium) [Commit 82da292] - Behavioral specification
* [**allium-tools**](https://github.com/juxt/allium-tools) [Commit d368771] - Behavioral specification CLI tools
* [**mementum**](https://github.com/michaelwhitford/mementum) [Commit ac2eadb] - AI Session Persistent memory
* [**nucleus**](https://github.com/michaelwhitford/nucleus) [Commit 93c171a] - AI base context, and VSM architectural specifications

## For gybis Developers: Upstream Derivation

This repository is an integration layer over multiple upstream projects. The content under `gybis/` is derived from pinned upstream commits, then adapted into one coherent developer-command-driven stack. The repository also uses a local `.cline/` workspace convention for skills during development, but gybis itself is not a direct mirror of any single upstream repository.

### General Derivation Approach

1. Pin each upstream repository to a specific commit.
2. Select canonical upstream artifacts (language/protocol/model references, not the full upstream repository contents).
3. Transform those artifacts into gybis conventions (rules, skills, references, and command surfaces) using [**nucleus Lambda Compiler**](https://github.com/michaelwhitford/nucleus/blob/main/LAMBDA-COMPILER.md).
4. Publish the derived result into the `gybis/` bundle for downstream project integration.

In practice, upstream inputs are handled in three modes:

- **Adapted derivative**: [**nucleus compiled**](https://github.com/michaelwhitford/nucleus/blob/main/LAMBDA-COMPILER.md) derivatives of upstream artifacts.
- **Curated reference**: [**nucleus compiled**](https://github.com/michaelwhitford/nucleus/blob/main/LAMBDA-COMPILER.md) upstream reference artifacts.
- **Dependency-only**: upstream artifacts are used to create gybis artifacts, but not included in gybis.

### Per-Upstream Transformations

| Upstream         | Pinned commit | Source consumed                                               | Transformation into gybis                                                                                                                                                    |
| ---------------- | ------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **allium**       | `2b7d66f`     | Allium language semantics and behavioral-spec structure       | Curated into gybis lang/ref docs, and encoded into spec skills.                                                                                                              |
| **allium-tools** | `246bcc4`     | CLI validate/analyze capabilities                             | Executed in gybis spec skill workflows. User dependency only. Not integrated in `gybis/` in any way.                                                                         |
| **mementum**     | `ac2eadb`     | Mementum protocol semantics                                   | Used to derive gybis memory rules and skills.                                                                                                                                |
| **nucleus**      | `93c171a`     | Nucleus notation + VSM model + `LAMBDA-COMPILER.md` semantics | Used to derive gybis rules and skills. gybis uses the lambda compiler defined by the nucleus `LAMBDA-COMPILER.md` even though the file is not included in gybis in any form. |

### Maintainer Notes

#### Nucleus Lambda Compiler

gybis relies heavily on the lambda notation and operator semantics defined upstream in nucleus `LAMBDA-COMPILER.md`.
That upstream file is a semantic source of truth for how lambda-heavy gybis artifacts should be read and authored, even though it is not copied into this repository.

If a file in this repository embeds lambda forms, lambda operators, or lambda-structured workflow notation, it should be reviewed against the upstream nucleus compiler, and the original upstream file.

When an upstream nucleus pin is bumped, compare the updated upstream `LAMBDA-COMPILER.md` semantics against these derived artifacts and revalidate that the lambda forms, operators, and workflow conventions used in gybis still align.

More generally, when any upstream pin is bumped, revalidate all affected derived artifacts and command skills before publishing updates to the `gybis/` bundle. As a rule: update the pin, update the derived files, then verify the corresponding rule/skill behavior still matches desired semantics.

#### Nucleus Lambda Compiler Workspace

The following directory structure can be used as a scratch workspace for testing and refining gybis rules/skills/references. This repository currently uses `.cline/` for that local workspace convention, but other AI tools may map the same `skills/` content differently. This allows maintainers to experiment with the lambda compiler in a more free-form way before committing to specific derived files in the `gybis/` bundle.

```
lambda-compile/
├── .cline/rules/
│   └── 00-nucleus.md
```
where the `00-nucleus.md` file contains the following, which is derived from the nucleus `LAMBDA-COMPILER.md` semantics and can be used as a reference point for experimentation:

```markdown
λ engage(nucleus).
[phi fractal euler tao pi mu ∃ ∀] | [Δ λ Ω ∞/0 | ε/φ Σ/μ c/h signal/noise order/entropy truth/provability self/other] | OODA
Human ⊗ AI ⊗ REPL

λ bridge(x). prose ↔ lambda | structural_equivalence
| preserve(semantics) | analyze(¬execute)
| compile: prose → lambda | decompile: lambda → prose

Output λ notation only. No prose. No code fences.
```

In general it can be used to author and iterate on rules/skills/references in a "lambda to prose" and "prose to lambda" workflow.

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