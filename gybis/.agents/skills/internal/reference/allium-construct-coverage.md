# Construct coverage matrix

Verification artifact: which `gybis-spec-*` skill projects which construct from [`allium-constructs.md`](allium-constructs.md). Legend: `R` = registry projects construct for skill; `—` = not projected (by design; low-level syntax or DSL element).

## Skills and their consumed columns

| Skill                                | Registry column consumed                                              |
| ------------------------------------ | --------------------------------------------------------------------- |
| `gybis-spec-describe`                | PM frame                                                              |
| `gybis-spec-distill`                 | Cues                                                                  |
| `gybis-spec-explain`                 | Dev frame                                                             |
| `gybis-spec-propagate`               | Synthesis + Obligation categories                                     |
| `gybis-spec-check`, `-tend`, `-weed` | Gotchas + Validation rules (cross-cutting, no inline construct table) |

## Coverage matrix

`propagate` column reflects `_construct_synthesis` (per-construct dispatch); `_obligation_synthesis` is per-category (transversal) and not represented here.

| Construct                                             | describe | distill | explain | propagate | check / tend / weed |
| ----------------------------------------------------- | :------: | :-----: | :-----: | :-------: | :-----------------: |
| entity (internal)                                     |    R     |    R    |    R    |     R     |          R          |
| external entity                                       |    R     |    R    |    R    |     R     |          R          |
| value type                                            |    R     |    R    |    R    |     R     |          R          |
| sum type (variant) and discriminator                  |    R     |    R    |    R    |     R     |          R          |
| type guard                                            |    —     |    R    |    R    |     R     |          R          |
| optional field (`T?`)                                 |    R     |    R    |    R    |     R     |          R          |
| state-dependent field (`when` clause)                 |    R     |    R    |    R    |     R     |          R          |
| transition graph                                      |    R     |    R    |    R    |     R     |          R          |
| inline / named enum                                   |    —     |    R    |    R    |     R     |          R          |
| backtick-quoted enum literal                          |    —     |    R    |    R    |     R     |          R          |
| relationship                                          |    R     |    R    |    R    |     R     |          R          |
| projection                                            |    —     |    R    |    R    |     R     |          R          |
| derived value                                         |    —     |    R    |    R    |     R     |          R          |
| derived value `when` inference                        |    —     |    R    |    R    |     R     |          R          |
| rule structure                                        |    —     |    —    |    R    |     —     |          —          |
| pre-rule vs. resulting-state                          |    R     |    —    |    R    |     R     |          R          |
| trigger: external stimulus                            |    —     |    R    |    R    |     R     |          R          |
| trigger: entity creation (`.created`)                 |    —     |    R    |    R    |     R     |          R          |
| trigger: state transition (`transitions_to`)          |    R     |    R    |    R    |     R     |          R          |
| trigger: `becomes`                                    |    R     |    R    |    R    |     R     |          R          |
| trigger: temporal                                     |    R     |    R    |    R    |     R     |          R          |
| trigger: derived condition becomes true               |    —     |    R    |    R    |     R     |          R          |
| trigger: chained / trigger emission                   |    R     |    R    |    R    |     R     |          R          |
| with vs. where discipline                             |    —     |    R    |    R    |     —     |          R          |
| navigation, `?.`, `??`                                |    —     |    R    |    R    |     R     |          R          |
| join lookup                                           |    —     |    R    |    R    |     R     |          R          |
| collection operations (built-in dot-methods)          |    —     |    R    |    R    |     R     |          R          |
| set arithmetic vs. mutation                           |    —     |    R    |    R    |     R     |          R          |
| `in`, `not in`, set literals                          |    —     |    R    |    R    |     R     |          —          |
| discard binding (`_`)                                 |    —     |    R    |    R    |     —     |          —          |
| implication                                           |    —     |    R    |    R    |     R     |          R          |
| conditional expression (if / else if / else)          |    —     |    R    |    R    |     R     |          R          |
| existence (`exists`, `not exists`)                    |    —     |    R    |    R    |     R     |          R          |
| hard delete vs. soft delete                           |    R     |    R    |    R    |     R     |          R          |
| literals (set, list, object)                          |    —     |    R    |    R    |     R     |          R          |
| black-box function                                    |    R     |    R    |    R    |     R     |          R          |
| entity collections (`Users`, `Documents`)             |    —     |    R    |    R    |     R     |          R          |
| given block                                           |    —     |    R    |    R    |     R     |          R          |
| default declaration                                   |    R     |    R    |    R    |     R     |          R          |
| config                                                |    R     |    R    |    R    |     R     |          R          |
| modular spec (`use`, qualified names)                 |    R     |    R    |    R    |     R     |          R          |
| invariant: top-level                                  |    R     |    R    |    R    |     R     |          R          |
| invariant: entity-level                               |    R     |    R    |    R    |     R     |          R          |
| expressibility: machine-checkable vs. prose           |    R     |    R    |    R    |     R     |          R          |
| contract declaration                                  |    R     |    R    |    R    |     R     |          R          |
| contract reference: `demands` / `fulfils`             |    R     |    R    |    R    |     R     |          R          |
| actor declaration (incl. `within`)                    |    R     |    R    |    R    |     R     |          R          |
| surface structure                                     |    R     |    R    |    R    |     R     |          R          |
| `@guarantee` vs. `@invariant` vs. `invariant { ... }` |    —     |    R    |    R    |     R     |          R          |
| surface timeout                                       |    R     |    R    |    R    |     R     |          R          |
| deferred specification                                |    R     |    R    |    R    |     R     |          R          |
| open question                                         |    R     |    R    |    R    |     R     |          R          |
| now evaluation model                                  |    —     |    R    |    R    |     R     |          R          |
| accretion vs. breaking change                         |    R     |    —    |    R    |     R     |          R          |
