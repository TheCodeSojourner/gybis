# Development with the Gybis Stack

This repository is structured for **development with the Gybis stack**, following a disciplined architecture-first methodology where durability, hierarchy, and behavioral truth guide every decision.

---

## Architecture Principles

- **Organize by durability** — Structure things by how long they will likely last.
- **Hierarchy of abstractions:** why > how > what.
- **Layered system:** S5 > S4 > S3 > S2 > S1 > specs > tests > code — stricter, more durable layers constrain looser, more transient ones.
- **No flat structures.** Everything has its place in the hierarchy.
- **Top-down only.** Higher layers constrain lower layers. Architecture (S5 ... S1) > Spec > Tests > Code.
- **No reverse dependencies.** Lower layers never constrain higher layers.
- **Drift surfaces automatically.** Drift is detected, surfaced, and halted.

---

## Specification Philosophy

Specifications describe code **behavior**, not implementation.

- **Elicit and Distill** architecture requirements into precise, checkable behavioral truths (`.allium`).
- **Do not implement before specifying.** Implementation details are replaceable; specifications are the durable operating system.
- **Workflow:** elicit → distill → check → propagate → tend → weed.
- **Governance flows:** AI is governed by formalized behavior; behavior is governed by architecture.

---

## Code as Replaceable Detail

- **`code ≡ replaceable_detail`** — Implementation is ephemeral and interchangeable.
- **`spec ≡ durable_os`** — Specifications are the operating system of the project.
- **`¬impl_before_spec`** — Never implement before specifying.
- **`¬test_before_spec`** — Never test before specifying.

---

## Memory System

Memory tracks the state of four domains: **architecture, specification, tests, and code**.

- **Session persistence:** Session `n+1` is proportional to the sum of all prior encodings from sessions `1..n`.
- **No knowledge loss** across sessions.
- **Commands:** `/gybis-memory-*` — encode state or restore from it.

---

## Session Memory Workflow

Each work session follows a structured lifecycle:

1. **Start:** Orient → Recall → Ready
2. **End:** Encode → Terminate

Every session guarantees **no knowledge loss**.

---

## Authority Model

- **Human commands come first.** The AI never takes initiative.
- **Human is the approval gate.** Every write operation requires human approval.
- **No autonomous AI actions.**

---

## Transparency

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

| Category      | Commands          |
| ------------- | ----------------- |
| Architecture  | `/gybis-arch-*`   |
| Memory        | `/gybis-memory-*` |
| Specification | `/gybis-spec-*`   |