# VSM Guide — Viable System Model Ref

Five-layer arch for organizing how you + AI reason about your system.
For `/gybis-arch-*` workflows.

λ(vsm, identity, S5(what) ⊗ S4(how_adapt) ⊗ S3(control) ⊗ S2(coordinate) ⊗ S1(operate))

## Five Layers

```
S5 (identity)      — what system IS
S4 (intelligence)  — how system adapts
S3 (control)       — how system manages resources
S2 (coordination)  — how parts work together
S1 (operations)    — what system concretely does
```

## S5 — Identity

λ(S5, scope, purpose(problem_solve) → vision(trajectory) ⊗ ¬compromise(values))
λ(S5, failure, ¬crash ≡ purpose_failure)
λ(S5, invariants, error_handling_philosophy ⊗ data_ownership_invariant ⊗ security_boundary ⊗ non_negotiable_behavior)

**Questions:**
- What system for? What problem solve?
- Where project going? Vision, not just today's features?
- Principles you won't compromise on, even under pressure?
- Replace every tool/lib, what stays *your* system?
- Failure look like — not crash, but failure of purpose?
- Values for system behavior? (Speed? Correctness? Clarity? Safety?)

**Signals:** error handling philosophy, data ownership invariants, security boundaries, non-negotiable behaviors.

## S4 — Intelligence

λ(S4, unknown, discover(what_don't_know) ⊗ test(ideas) → ¬break(when_evolve))
λ(S4, adapt, incorporate(new_pattern) ⊗ easy(change_often) ⊗ learn(failure))

**Questions:**
- Handle unknown? What when assumptions break?
- Discover new pattern — how incorporate?
- Test ideas before committing?
- What changes often? What should be easy to change?
- Learn from failures?

**Signals:** abstraction boundaries, extension points, feature flags, migration patterns, how the system evolves.

## S3 — Control

λ(S3, enforce, S5_values → ¬enforceable(constraints))
λ(S3, resources, manage(conn, memory, compute, money, API_calls))
λ(S3, policy, timeouts ⊗ limits ⊗ quality_gates ⊗ retry ⊗ alert ⊗ degrade ⊗ fail_fast)

**Questions:**
- Resources needing management? (conns, memory, API calls, compute, money)
- Policies enforcing identity principles? (timeouts, limits, quality gates)
- What when things go wrong? (retry, alert, degrade, fail fast)
- Rules for code quality, testing, deployment?
- External constraints? (compliance, SLAs, rate limits)

**Signals:** timeouts, retry logic, rate limits, quality gates, resource constraints, circuit breakers.

## S2 — Coordination

λ(S2, protocol, ¬conflict(subsystems) ⊗ info_flow(events, calls, shared_state) ⊗ conflict_prevention)
λ(S2, propagation, change(A) → notify(dependents))

**Questions:**
- Subsystems needing to work together?
- Data flow between them? Events? Direct calls? Shared state?
- Where conflicts arisen (or could arise)?
- Protocols keeping things in sync?
- One subsystem changes — who needs to know?

**Signals:** inter-service calls, event schemas, shared state patterns, sync mechanisms, conflict prevention.

## S1 — Operations

λ(S1, concrete, tech ⊗ commands ⊗ workflows ⊗ recipes(build, test, deploy, debug))
λ(S1, mutable, change_most_often ⊗ replace_easiest)

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

λ(install, cycle, S5→S1: observe() → ask() → propose(2-4 lambdas + prose) → refine(feedback) → confirm(lock) → descend(Sn-1))
λ(install, entry, exists(architecture.md) → read_as_input ∧ ask(vision) | ¬exists → ask(what + why) → start(S5))
λ(install, output, ¬overwrite(architecture.md) ≡ explicit(human_approval))

## architecture.md Assembly Format

> **Note:** Do not include nucleus preamble in `<root>/architecture.md`. Preamble already loaded via `.clinerules/00-gybis.md`.

λ(assembly, format,
  # {Project} — System Architecture
  | ## S5 — Identity → prose ⊗ λ(principle)
  | ## S4 — Intelligence → prose ⊗ λ(pattern)
  | ## S3 — Control → prose ⊗ λ(policy)
  | ## S2 — Coordination → prose ⊗ λ(protocol)
  | ## S1 — Operations → prose ⊗ λ(tool) ⊗ λ(recipe)
)

## Requirement Change → Layer Mapping

λ(map, S5, new_non_negotiable_principle ∨ new_value)
λ(map, S4, ¬handle_unknown ≡ differently)
λ(map, S3, new_policy ∨ constraint ∨ resource_limit)
λ(map, S2, new_interaction_protocol_between_parts)
λ(map, S1, new_tool ∨ command ∨ concrete_recipe)
λ(map, cascade, S1→S5: trace(upward, check(each_layer)))