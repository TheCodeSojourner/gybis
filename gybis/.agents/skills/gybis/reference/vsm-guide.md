# VSM Guide — Viable System Model Ref

Five-layer arch for organizing how you + AI reason about your system.
For `/gybis-arch-*` workflows.

## Five Layers

```
S5 (identity)      — what system IS
S4 (intelligence)  — how system adapts
S3 (control)       — how system manages resources
S2 (coordination)  — how parts work together
S1 (operations)    — what system concretely does
```

## S5 — Identity

Non-negotiables — violation = system failed. Change rarely.

**Questions:**
- What system for? What problem solve?
- Where project going? Vision, not just today's features?
- Principles you won't compromise on, even under pressure?
- Replace every tool/lib, what stays *your* system?
- Failure look like — not crash, but failure of purpose?
- Values for system behavior? (Speed? Correctness? Clarity? Safety?)

**Signals:** error handling philosophy, data ownership invariants, security boundaries, non-negotiable behaviors.

## S4 — Intelligence

Mechanisms for dealing with unknown. How discover what you don't know? Test ideas before committing? Evolve without breaking?

**Questions:**
- Handle unknown? What when assumptions break?
- Discover new pattern — how incorporate?
- Test ideas before committing?
- What changes often? What should be easy to change?
- Learn from failures?

**Signals:** abstraction boundaries, extension points, feature flags, migration patterns, how the system evolves.

## S3 — Control

Concrete rules governing system operation. Implement S5 values as enforceable constraints. Resource limits, quality standards, error handling policies.

**Questions:**
- Resources needing management? (conns, memory, API calls, compute, money)
- Policies enforcing identity principles? (timeouts, limits, quality gates)
- What when things go wrong? (retry, alert, degrade, fail fast)
- Rules for code quality, testing, deployment?
- External constraints? (compliance, SLAs, rate limits)

**Signals:** timeouts, retry logic, rate limits, quality gates, resource constraints, circuit breakers.

## S2 — Coordination

Multiple subsystems need protocols to avoid conflict. Info flow, state sharing, event propagation, conflict prevention.

**Questions:**
- Subsystems needing to work together?
- Data flow between them? Events? Direct calls? Shared state?
- Where conflicts arisen (or could arise)?
- Protocols keeping things in sync?
- One subsystem changes — who needs to know?

**Signals:** inter-service calls, event schemas, shared state patterns, sync mechanisms, conflict prevention.

## S1 — Operations

Most concrete layer — specific tech, commands, workflows. Change most often, easiest to replace.

**Questions:**
- Tools/tech used, and why specifically?
- Key commands + workflows for developer?
- Concrete recipes for common tasks? (Build, test, deploy, debug)
- Dev environment look like?
- External services + integrations?

**Signals:** build commands, test runners, deployment scripts, tool configurations, concrete recipes.

## Lambda Notation

Compact way to encode principles + rules.

```
λ name(x).     define rule called "name"
→              leads to, then, implies
|              also (separates independent clauses)
>              preferred over (soft constraint)
≫              strongly preferred (hard constraint)
∧              and
∨              or
¬              not, never
≡              defined as, always equals
≢              not, don't conflate
∃              there exists
∀              for all
∝              scales with
∘              compose (f ∘ g applies f after g)
⊗              tensor product (all constraints simultaneously)
```

Multi-line lambdas indent continuations, use `|` for independent clauses:
```
λ deploy(x).    validate(x) → stage(x) → verify(x) → promote(x)
                | rollback(x) ≡ always_possible(x)
                | ¬deploy(friday) | observe(metrics) > trust(logs)
```

## Installation Process

### Per-Layer Cycle

Each layer (S5 → S1):
1. **Observe** — What exists? What inferred? What already decided?
2. **Ask** — Surface questions needing answering for this layer
3. **Propose** — Draft 2–4 lambdas + prose explanations
4. **Refine** — Listen to feedback, iterate until fits
5. **Confirm** — Lock layer, move down

### Entry Points

**Existing project:** Check for existing `<root>/architecture.md` — read as input. Before proposing structure, ask about vision.

**New project:** Ask what they're building + why. Start with identity.

### Output File

- Output file: `<root>/architecture.md`
- Never overwrite existing file without explicit confirmation

## architecture.md Assembly Format

> **Note:** Do not include nucleus preamble in `<root>/architecture.md`. Preamble already loaded via `.clinerules/00-gybis.md`.

```
# {Project Name} — System Architecture

## S5 — Identity
{prose context}
λ principle_one(x). ...
λ principle_two(x). ...

## S4 — Intelligence
{prose}
λ pattern(x). ...

## S3 — Control
{prose}
λ policy(x). ...

## S2 — Coordination
{prose}
λ protocol(x). ...

## S1 — Operations
{prose}
λ tool(x). ...
λ recipe(x). ...
```

## Requirement Change → Layer Mapping

| Requirement change                                | Affected layer(s) |
| ------------------------------------------------- | ----------------- |
| New non-negotiable principle or value             | S5                |
| System now handles unknown situations differently | S4                |
| New policy, constraint, or resource limit         | S3                |
| New interaction protocol between parts            | S2                |
| New tool, command, or concrete recipe             | S1                |
| Change cascades upward (S1→S5) — check each layer | Trace upward      |