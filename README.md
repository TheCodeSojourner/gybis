# gybis /JY-bis/ 

<img src="gybis-logo.png" alt="gybis logo" width="110" style="margin-bottom: 1em;" />



**A Developer-Command-Driven AI-Assisted Spec-Driven Software Development (SDD) Stack.**

## Obligatory Word Definitions

**Gyre** /jahy<sup>uh</sup>r/

* a large-scale, coherent, rotating or spiraling flow

**Orbis** /ˈɔr.bis/

* a closed or periodic trajectory in a dynamical system, representing the set of all points reachable from an initial point under the system's update rules

## Overview

**gybis** contains a collection of [Cline](https://github.com/cline/cline) artifacts (e.g., rules, workflows), which when integrated into a new/existing repository, adds a **Developer-Command-Driven AI-Assisted Spec-Driven Software Development (SDD) Stack** to the new/existing repository. 

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

## Versioning

gybis tries to follow [Clojure's](https://github.com/clojure/clojure) versioning philosophy by prioritizing stability, backward compatibility, and minimal breakage over rapid evolution or strict adherence to semantic versioning ([SemVer](https://semver.org/)). 

- **Strong emphasis on backward compatibility**: Development will take a measured, thoughtful approach to evolution. Breaking changes will be avoided whenever possible. Releases will focus on enhancements, performance, and new capabilities while making every attempt to preserve existing behavior.
- **No fixed roadmap**: Development is open-ended. Alpha/beta/RC phases allow visibility into changes, but final releases will be are very stable. Deprecations will be handled carefully.

## For gybis Users

### Integrating gybis into Your Repository

Copy the contents of the `gybis/` directory into the root of your target repository:

```bash
cp -r gybis/. /path/to/your-repo/
```

After copying, your target repository should contain:

```
your-repo/
  .agents/             ← skill commands (/gybis-arch-*, /gybis-spec-*, /gybis-memory-*)
  .clinerules/         ← gybis-specific rules loaded by Cline
  mementum/            ← session memory template (state.md, memories/, knowledge/)
  specs/               ← placeholder for your project's .allium behavioral specs
  GYBIS-README.md      ← gybis reference and instructions
```

Once integrated, use [Cline](https://github.com/cline/cline) and start with `/gybis-arch-elicit` for a new repository, or `/gybis-spec-distill` for an existing repository. Additionally, you can use `/gybis` for general guidance on which commands are available and what they do.

### Installing the Allium CLI

The `allium` CLI is required for spec validation and analysis commands (`/gybis-spec-check`, `/gybis-spec-distill`, etc.). It must be installed on your system separately.

See the [allium-tools repository](https://github.com/juxt/allium-tools) for installation instructions.

## Upstream Repositories

* [**allium**](https://github.com/juxt/allium) [Commit 82da292] - Behavioral specification
* [**allium-tools**](https://github.com/juxt/allium-tools) [Commit d368771] - Behavioral specification CLI tools
* [**mementum**](https://github.com/michaelwhitford/mementum) [Commit ac2eadb] - AI Session Persistent memory
* [**nucleus**](https://github.com/michaelwhitford/nucleus) [Commit 93c171a] - AI base context, and VSM architectural specifications

## For gybis Developers: Upstream Derivation

This repository is an integration layer over multiple upstream projects. The content under `gybis/` is derived from pinned upstream commits, then adapted into one coherent developer-command-driven stack. gybis is not a direct mirror of any single upstream repository.

### General Derivation Approach

1. Pin each upstream repository to a specific commit.
2. Select canonical upstream artifacts (language/protocol/model references, not the full upstream repository contents).
3. Transform those artifacts into gybis conventions (rules, skill workflows, references, and command surfaces).
4. Publish the derived result into the `gybis/` bundle for downstream project integration.

In practice, upstream inputs are handled in three modes:

- **Adapted derivative**: upstream concepts are rewritten into gybis-oriented docs/rules.
- **Curated reference**: upstream semantics are packaged as local references for skill execution.
- **Dependency-only**: upstream tooling is invoked by workflows but not mirrored as static files.

### Per-Upstream Transformations

| Upstream         | Pinned commit | Source consumed                                               | Transformation into gybis                                                                                                                                                                                                                                   | Derived content                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ---------------- | ------------- | ------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **nucleus**      | `93c171a`     | Nucleus notation + VSM model + `LAMBDA-COMPILER.md` semantics | Distilled into gybis rule preambles, lambda-structured workflows, and architecture workflow guidance for `/gybis-arch-*` commands. gybis tracks the lambda semantics defined by nucleus `LAMBDA-COMPILER.md` even though that file is not vendored locally. | `.clinerules/01-nucleus-preamble.md`, `gybis/.clinerules/00-gybis.md`, `gybis/.agents/skills/gybis/reference/vsm-guide.md`, `gybis/.agents/skills/gybis-arch-elicit/SKILL.md`, `gybis/.agents/skills/gybis-arch-distill/SKILL.md`, `gybis/.agents/skills/gybis-arch-tend/SKILL.md`, `gybis/.agents/skills/gybis-arch-weed/SKILL.md`, `gybis/.agents/skills/gybis-arch-explain/SKILL.md`, `gybis/.agents/skills/gybis-arch-describe/SKILL.md`                                                                                                                                                                                                                                                                                                                                                                            |
| **mementum**     | `ac2eadb`     | Mementum protocol semantics                                   | Adapted into gybis memory rules and operating model with explicit template-vs-active memory separation.                                                                                                                                                     | `.clinerules/02-mementum.md`, `gybis/.clinerules/01-mementum.md`, `gybis/.agents/skills/gybis-memory-orient/SKILL.md`, `gybis/.agents/skills/gybis-memory-recall/SKILL.md`, `gybis/.agents/skills/gybis-memory-session-terminate/SKILL.md`, `gybis/.agents/skills/gybis-memory-store/SKILL.md`, `gybis/.agents/skills/gybis-memory-synthesize/SKILL.md`                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| **allium**       | `82da292`     | Allium language semantics and behavioral-spec structure       | Curated into local language/reference docs and encoded into spec skill workflows used for elicitation, distillation, propagation, and drift handling.                                                                                                       | `gybis/.agents/skills/gybis/reference/allium-actioning-findings.md`, `gybis/.agents/skills/gybis/reference/allium-assessing-specs.md`, `gybis/.agents/skills/gybis/reference/allium-language-reference.md`, `gybis/.agents/skills/gybis/reference/allium-library-spec-signals.md`, `gybis/.agents/skills/gybis/reference/allium-patterns.md`, `gybis/.agents/skills/gybis-spec-check/SKILL.md`, `gybis/.agents/skills/gybis-spec-describe/SKILL.md`, `gybis/.agents/skills/gybis-spec-distill/SKILL.md`, `gybis/.agents/skills/gybis-spec-elicit/SKILL.md`, `gybis/.agents/skills/gybis-spec-explain/SKILL.md`, `gybis/.agents/skills/gybis-spec-propagate/SKILL.md`, `gybis/.agents/skills/gybis-spec-tend/SKILL.md`, `gybis/.agents/skills/gybis-spec-weed/SKILL.md`, `gybis/.agents/skills/gybis-arch-weed/SKILL.md` |
| **allium-tools** | `d368771`     | CLI validation/analyze capabilities                           | Used as workflow gates (`allium-check`, `allium-analyse`) inside gybis spec skills; tooling is dependency-only and not mirrored as standalone docs/files in `gybis/`.                                                                                       | `gybis/.agents/skills/gybis-spec-check/SKILL.md`, `gybis/.agents/skills/gybis-spec-distill/SKILL.md`, `gybis/.agents/skills/gybis-spec-elicit/SKILL.md`, `gybis/.agents/skills/gybis-spec-tend/SKILL.md`, `gybis/.agents/skills/gybis-spec-weed/SKILL.md`, `gybis/.agents/skills/gybis-spec-propagate/SKILL.md`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |

### Maintainer Notes

#### Nucleus Lambda Compiler Contract

gybis relies heavily on the lambda notation and operator semantics defined upstream in nucleus `LAMBDA-COMPILER.md`.
That upstream file is a semantic source of truth for how lambda-heavy gybis artifacts should be read and authored, even though it is not copied into this repository.

Developers should treat all lambda-bearing local artifacts as downstream expressions of that contract. In this repository, that includes at least:

- `.clinerules/01-nucleus-preamble.md`
- `.clinerules/02-mementum.md`
- `gybis/.clinerules/00-gybis.md`
- `gybis/.clinerules/01-mementum.md`
- `gybis/.agents/skills/gybis/reference/vsm-guide.md`
- `gybis/.agents/skills/gybis/reference/allium-actioning-findings.md`
- `gybis/.agents/skills/gybis/reference/allium-assessing-specs.md`
- `gybis/.agents/skills/gybis/reference/allium-language-reference.md`
- `gybis/.agents/skills/gybis/reference/allium-library-spec-signals.md`
- `gybis/.agents/skills/gybis-arch-elicit/SKILL.md`
- `gybis/.agents/skills/gybis-arch-distill/SKILL.md`
- `gybis/.agents/skills/gybis-arch-tend/SKILL.md`
- `gybis/.agents/skills/gybis-arch-weed/SKILL.md`
- `gybis/.agents/skills/gybis-arch-explain/SKILL.md`
- `gybis/.agents/skills/gybis-arch-describe/SKILL.md`
- `gybis/.agents/skills/gybis-memory-orient/SKILL.md`
- `gybis/.agents/skills/gybis-memory-recall/SKILL.md`
- `gybis/.agents/skills/gybis-memory-session-terminate/SKILL.md`
- `gybis/.agents/skills/gybis-memory-store/SKILL.md`
- `gybis/.agents/skills/gybis-memory-synthesize/SKILL.md`
- `gybis/.agents/skills/gybis-spec-check/SKILL.md`
- `gybis/.agents/skills/gybis-spec-describe/SKILL.md`
- `gybis/.agents/skills/gybis-spec-distill/SKILL.md`
- `gybis/.agents/skills/gybis-spec-elicit/SKILL.md`
- `gybis/.agents/skills/gybis-spec-explain/SKILL.md`
- `gybis/.agents/skills/gybis-spec-propagate/SKILL.md`
- `gybis/.agents/skills/gybis-spec-tend/SKILL.md`
- `gybis/.agents/skills/gybis-spec-weed/SKILL.md`
- `.agents/skills/gybis-mementum-orient/SKILL.md`
- `.agents/skills/gybis-mementum-recall/SKILL.md`
- `.agents/skills/gybis-mementum-session-terminate/SKILL.md`
- `.agents/skills/gybis-mementum-store/SKILL.md`
- `.agents/skills/gybis-mementum-synthesize/SKILL.md`

If a file in this repository embeds lambda forms, lambda operators, or lambda-structured workflow notation, it should be reviewed against the upstream nucleus contract as part of that same surface area.

When an upstream nucleus pin is bumped, compare the updated upstream `LAMBDA-COMPILER.md` semantics against these derived artifacts and revalidate that the lambda forms, operators, and workflow conventions used in gybis still align.

More generally, when any upstream pin is bumped, revalidate all affected derived artifacts and command workflows before publishing updates to the `gybis/` bundle. As a rule: update the pin, update the derived files, then verify the corresponding skill/rule behavior still matches upstream semantics.

#### Nucleus Lambda Compiler Workspace

The following directory structure can be used with `cline` as a scratch workspace for testing and refining lambda forms, operators, and workflow structures that will eventually be embedded into gybis artifacts. This allows maintainers to experiment with the lambda contract in a more free-form way before committing to specific derived files in the `gybis/` bundle.

```
lambda-compile/
├── .clinerules/
│   └── 01-nucleus-preamble.md
```
where the `01-nucleus-preamble.md` file contains the following, which is derived from the nucleus `LAMBDA-COMPILER.md` semantics and can be used as a reference point for experimentation:

```markdown
λ engage(nucleus).
[phi fractal euler tao pi mu ∃ ∀] | [Δ λ Ω ∞/0 | ε/φ Σ/μ c/h signal/noise order/entropy truth/provability self/other] | OODA
Human ⊗ AI ⊗ REPL

λ bridge(x). prose ↔ lambda | structural_equivalence
| preserve(semantics) | analyze(¬execute)
| compile: prose → lambda | decompile: lambda → prose

Output λ notation only. No prose. No code fences.
```

In general: 

- it can be used to author and iterate on agent/prompt rules for a “lambda compile” workflow, with the core contract living in 01-nucleus-preamble.md. 
- Second, it can act as a minimal sandbox for experimenting with a notation or transformation design before any implementation exists. 
- Third, it can serve as the initial repository shell for a future compiler, interpreter, or REPL-oriented tool that translates between prose and a lambda-style formal notation.
  - **compile**: take informal natural-language meaning and encode it into a formal lambda-style notation.
  - **decompile**: take the formal notation and recover readable prose that preserves the same semantics.

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
