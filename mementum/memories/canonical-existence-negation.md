---
type: Insight
symbol: 💡
title: Canonical existence notation is ∃ / ¬∃ (∅ is a separate canonical operator)
status: open
related: absence-operator-selection-rule
---

Upstream authority: existence is canonically `∃`, negated `¬∃`. The empty-set operator `∅` is *also* canonical but distinct. The verbal `exists` is not an operator at all.

Sources (github.com/michaelwhitford/nucleus):
- **Preamble** — `[phi fractal euler tao pi mu ∃ ∀]`; symbols only, no verbal `exists`.
- **SYMBOLIC_FRAMEWORK.md** — ontology table lists `∃ (exists)`; "exists" is the gloss, not an operator.
- **EBNF.md** — `negation = "¬" , term`; `∃`/`∀` are excluded from the formal term grammar, so they are preamble-level notation negated by `¬`.
- **VSM.md** — operational form is uniformly `∃x` / `¬∃x`.
- **SYSTEM_DESIGN.md** — the full operator table. Lists `∀`/`∃` and `∅` ("Empty set / no result"); it does not list a verbal `exists`.

| Form                                      | Status                   |
| ----------------------------------------- | ------------------------ |
| `∃` / `¬∃`                                | canonical                |
| `∅`                                       | canonical (different op) |
| `exists(...)` / `¬exists` (operator form) | drift                    |
| `X_exists` / `exists?` (identifier)       | fine                     |
| `not exists X`                            | Allium DSL — never touch |

`∅` and `¬∃` differ: `¬∃x` = "x does not exist"; `x = ∅` = "the set is empty". Both canonical; choose by presence-symmetry (see related).
