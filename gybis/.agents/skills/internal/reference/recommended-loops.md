# Recommended loops

Allium keeps three artifacts in agreement: the spec (intended behavior), the tests (the executable contract), and the code (the implementation). Productive work is driving those three to convergence and keeping them there as things change. This is a loop, not a pipeline.

This reference focuses on the gybis spec loop.

Two practical entry conditions:
- spec-ready: a spec already exists and you refine it with `/gybis-spec-tend`
- code-first: derive a candidate spec from code with `/gybis-spec-distill`

Note: gybis does not provide `/gybis-spec-elicit`. If no spec exists yet, create it through the architecture flow first (`/gybis-arch-elicit` -> `/gybis-arch-propagate`), then continue with this spec loop.

## The loop: gather context -> take action -> verify -> repeat

An autonomous agent works a task as a recursive goal: it gathers context, takes action, verifies the result, and repeats until the goal is met. The phase that decides quality is verification, a feedback signal the loop can trust. Allium strengthens the two hardest phases, context and verification:

| Phase          | What you do                                                                                          | What Allium contributes                                                                             |
| -------------- | ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| Gather context | spec-ready `/gybis-spec-tend`, or code-first `/gybis-spec-distill` -> the spec                       | The spec is the context, and it persists across sessions; no drift versus re-reading code each time |
| Take action    | `/gybis-spec-propagate` -> tests, then implement code                                                | Tests are generated from the spec as the contract; code is written against them                     |
| Verify         | run tests (must pass), then `/gybis-spec-weed` for spec<->code alignment, then CLI structural checks | A behavioral pass/fail signal, not merely unit tests green                                          |
| Repeat         | iterate until convergence is met                                                                     | The convergence invariant below is the goal                                                         |

The point is not command order; it is that an autonomous loop is only as good as the signal it verifies against. Allium supplies a behavior-level signal: tests projected from intent plus alignment and structural checks.

## The convergence invariant

The loop is done for a unit of work when:

- tests pass: implementation satisfies the spec's obligations
- `/gybis-spec-weed` reports no divergence: spec and code agree about behavior
- no open questions remain: ambiguities were resolved, not guessed
- (code-first only) a fresh `/gybis-spec-distill` pass surfaces nothing new

While any of these is false, there is work left.

Convergence governs the whole loop, not only verify. Ordinary test-driven coding has an inner arc, implement <-> run tests, that drives code toward a fixed contract. Allium adds an outer arc: verify can feed back into context, not only action. If `/gybis-spec-weed` shows the spec (not code) is wrong, or a test reveals ambiguity, re-enter gather-context with `/gybis-spec-tend` (or ask the human), then re-run `/gybis-spec-propagate`.

## The skills as loop operators

| Skill                   | Moves         | Direction                             |
| ----------------------- | ------------- | ------------------------------------- |
| `/gybis-spec-distill`   | code -> spec  | write spec (backward)                 |
| `/gybis-spec-propagate` | spec -> tests | project spec into contract            |
| implement               | tests -> code | ordinary coding, not an Allium skill  |
| `/gybis-spec-weed`      | code <-> spec | reconcile divergence either direction |
| `/gybis-spec-tend`      | edits spec    | re-enter the loop after change        |

The implementation step is plain coding. Allium produces the spec and tests; those hold hand-written code to specified behavior.

## Loop A: spec-first (forward, from intent)

When to use: new feature or greenfield work where a spec is already available.

```
until (tests pass and weed clean and no open questions):
  1) /gybis-spec-tend -> spec (or confirm existing spec still captures intent)
  2) /gybis-spec-propagate -> tests for new behavior
  3) run new tests and confirm they fail (red)
  4) implement from spec + failing tests
  5) run tests
     - failing: fix code, continue
     - test appears wrong: /gybis-spec-tend spec, then /gybis-spec-propagate
  6) /gybis-spec-weed for spec<->code alignment
     - divergence: reconcile; if intent changed, /gybis-spec-tend
     - open question: ask human, then /gybis-spec-tend
```

Never weaken a generated test to pass. If a generated test appears wrong, fix spec and re-propagate.

## Loop B: code-first (backward, from existing code)

When to use: existing/unfamiliar codebase you need to capture, verify, or change safely.

```
until (distill finds nothing new and failures triaged and weed clean):
  1) /gybis-spec-distill <area> -> candidate spec of current behavior
  2) review intended vs accidental behavior
  3) /gybis-spec-propagate -> tests
  4) run tests against existing code
     - pass: behavior captured
     - fail: triage code bug vs spec wrong
       * code bug: fix code
       * spec wrong: /gybis-spec-tend, then /gybis-spec-propagate
  5) /gybis-spec-weed to reconcile remaining divergence
  6) expand to next area and repeat
```

In code-first, passing tests against existing code is the expected confirmation signal; failures are information to classify.

## Running the loop autonomously

Per tick:
1. Advance spec: `/gybis-spec-distill` on first tick for code-first, or `/gybis-spec-tend` for spec-ready work; use `/gybis-spec-tend` thereafter if needed.
2. Run `/gybis-spec-propagate` if spec changed.
3. Implement/fix code toward green (spec-first) or triage failures (code-first).
4. Run `/gybis-spec-weed` for alignment.
5. Re-evaluate convergence invariant.

Exit conditions:
- convergence invariant satisfied, or
- bounded iteration budget exhausted.

Guardrails:
- confirm generated tests fail before implementing in spec-first flow
- never edit generated tests to force green
- escalate ambiguity to human instead of guessing
- honor config parameters in code, avoid magic numbers where spec uses config
- fix code, not contract, when spec is correct

Track across ticks: test pass/fail counts, `/gybis-spec-weed` verdict, open question count, and whether latest `/gybis-spec-distill` surfaced anything new.

## Produce-code prompt

Implement behavior in `<spec>.allium`. The generated tests in `<tests>` are the contract and must pass. Follow the spec's `when`/`requires`/`ensures` semantics. Do not weaken or edit tests to go green; if a test looks wrong, stop and revisit spec.

## Related

- [Assessing specs](./allium-assessing-specs.md)
- [Actioning findings](./allium-actioning-findings.md)
