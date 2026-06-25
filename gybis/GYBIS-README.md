# gybis /JY-bis/ 

**A Developer-Command-Driven AI-Assisted Spec-Driven Software Development (SDD) Stack.**

## Obligatory Word Definitions

**Gyre** /jahy<sup>uh</sup>r/

* a large-scale, coherent, rotating or spiraling flow

**Orbis** /ˈɔr.bis/

* a closed or periodic trajectory in a dynamical system, representing the set of all points reachable from an initial point under the system's update rules

## Development with the Gybis Stack

This repository is structured for **development with the Gybis stack**, following a disciplined architecture-first methodology where durability, hierarchy, and behavioral truth guide every decision.

When gybis is installed into a target repository, its command implementations are shipped in the bundled `.agents/skills/` directory.

gybis is a **developer-command-driven, AI-assisted Spec-Driven Development (SDD) stack**. It is designed to help you define architecture and behavioral specifications first, then implement and validate code against those specifications with full human oversight.

## What gybis Is and Why It Exists

In gybis, specifications are durable and implementation is replaceable.

- **Spec-Driven Development (SDD):** Architecture and behavioral specifications define what the system is and does, and implementation follows those constraints.
- **Human-controlled AI assistance:** AI supports analysis, authoring, and validation, but humans stay in control of decisions and approvals.
- **Durable truth model:** Architecture and behavioral specifications are the source of truth; code and tests must align to them.

## Installing the Allium CLI

The `allium` CLI is required for spec validation and analysis commands (`/gybis-spec-check`, `/gybis-spec-distill`, etc.). It must be installed on your system separately.

See the [allium-tools repository](https://github.com/juxt/allium-tools) for installation instructions.

## Quick Start

- **New repository:** Start with `/gybis-arch-elicit` to establish architecture constraints.
- **Existing repository:** Start with `/gybis-spec-distill` to extract behavioral specifications from current implementation, then use `/gybis-arch-distill` to derive architecture from those specifications.
- **Need guidance:** Run `/gybis-help` to see available commands.

---

## Architecture Principles

- **Organize by durability** — Structure things by how long they will likely last.
- **Hierarchy of abstractions:** why > what > how.
- **Layered system:** S5 > S4 > S3 > S2 > S1 > specs > tests > code — stricter, more durable layers constrain looser, more transient ones.
  S5..S1 are architectural layers; together they constrain specifications, which then constrain tests, which then constrain code.
- **No flat structures.** Everything has its place in the hierarchy.
- **Top-down only.** Higher layers constrain lower layers. Architecture (S5 ... S1) > Specs > Tests > Code.
- **No reverse dependencies.** Lower layers never constrain higher layers.
- **Drift surfaces automatically.** When code, tests, or behavior diverge from architecture/specification constraints, drift is detected, surfaced, and halted.

---

## Architecture Philosophy

Architecture describes system-level constraints that drive behavior specifications that drive tests, that drive code.

- **Constrain top-down.** Architecture governs specification, tests, and code, not the reverse.
- **Governance flow:** Architecture, specification, tests and implementation must remain aligned.

### New Repository

For a new repository, start with `/gybis-arch-elicit` to establish durable architectural constraints with the developer, then derive behavior specifications from the architecture with `/gybis-arch-propagate`, and finally derive code and tests from the behavior specifications with `/gybis-spec-propagate`.

### Existing Repository

For an existing repository, start with `/gybis-spec-distill` to create a set of behavior specifications from tests and code, then establish durable architectural constraints with `/gybis-arch-distill`.

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

Memory tracks the state of four domains: **architecture, specification, tests, and code**.

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
architecture > specification > tests > code
```

- **No bypassing the hierarchy.**
- **No specification before architecture.**
- **No testing before specification.**
- **No implementation before testing.**
- **Higher layers constrain lower layers.**
- **Violations surface immediately and halt progress.**

---

## Commands

| Skill Name                                                       | Description                                   |
| ---------------------------------------------------------------- | --------------------------------------------- |
| `/gybis-arch-describe` (`/ga-describe`)                          | Describe arch in non-tech prose or markdown   |
| `/gybis-arch-distill` (`/ga-distill`)                            | Create initial arch from specs                |
| `/gybis-arch-elicit` (`/ga-elicit`)                              | Create initial arch with human                |
| `/gybis-arch-explain` (`/ga-explain`)                            | Explain arch in dev prose or markdown         |
| `/gybis-arch-propagate` (`/ga-propagate`)                        | Create initial specs from arch                |
| `/gybis-arch-tend` (`/ga-tend`)                                  | Update arch with human                        |
| `/gybis-arch-weed` (`/ga-weed`)                                  | Upsert arch/specs from diffs with human       |
| `/gybis-fini`                                                    | CRUD memory before terminate                  |
| `/gybis-init`                                                    | Initialize gybis AI context                   |
| `/gybis-memory-orient` (`/gm-orient`)                            | Restore prev AI context                       |
| `/gybis-memory-recall {topic}` (`/gm-recall {topic}`)            | Recall topic/summarize-latest                 |
| `/gybis-memory-store {insight}` (`/gm-store {insight}`)          | Store insight, or prompt for one              |
| `/gybis-memory-synthesize` (`/gm-synthesize`)                    | Synthesize knowledge                          |
| `/gybis-spec-check` (`/gs-check {concern\|domain\|all}`)         | Check/Update syntax until valid               |
| `/gybis-spec-describe` (`/gs-describe {concern\|domain\|all}`)   | Describe in non-tech prose or markdown        |
| `/gybis-spec-distill` (`/gs-distill`)                            | Create initial specs from code/tests          |
| `/gybis-spec-explain` (`/gs-explain {concern\|domain\|all}`)     | Explain in dev prose or markdown              |
| `/gybis-spec-propagate` (`/gs-propagate {concern\|domain\|all}`) | Create initial code/tests                     |
| `/gybis-spec-tend` (`/gs-tend`)                                  | Update specs with human                       |
| `/gybis-spec-weed` (`/gs-weed`)                                  | Upsert specs/code-tests from diffs with human |

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

## gybis

This project incorporates ideas, code, and/or structure from
[gybis](https://github.com/TheCodeSojourner/gybis)
by Paul Whittington.

Original project:
https://github.com/TheCodeSojourner/gybis

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