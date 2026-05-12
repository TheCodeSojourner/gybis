# Allium CLI — Actioning Findings

Read JSON from `allium check`. Turn into human issues. Decide fix.

---

## Canonical Sequence

```bash
allium check <file> 
```
---

## Reading check Diagnostics

Parse `diagnostics` array. Group by severity:

1. **`error`** — blocks write; fix before save
2. **`warning`** — human decision (fix or accept)
3. **`info`** — advisory; show human, no block

**Present each as:**

```
[E/W/I] {code} at line {line}: {message}
  → Suggested fix: <interpretation>
```

**Common diagnostics:**

| Code pattern              | Category             | Ask human                                                                     |
| ------------------------- | -------------------- | ----------------------------------------------------------------------------- |
| Missing `ensures:`        | Incomplete rule      | "Rule has no outcome — what happens after [trigger]?"                         |
| Undefined field reference | Missing entity field | "Field `[name]` used in rule but not on [Entity]. Add it?"                    |
| Contradictory `requires:` | Spec conflict        | "Rules [A] and [B] have conflicting preconditions on [field]. Which correct?" |
| `when`-clause obligation  | State field gap      | "Transitioning [Entity] to [state] needs field `[name]`. Which rule sets it?" |
| No `when:` on rule        | Unreachable rule     | "Rule [Name] has no trigger — when should it fire?"                           |
| Stale `traces:`           | Implementation drift | "Traces reference [file] gone. Update or remove?"                             |
| Unused `use` import       | Dead import          | "Module [alias] imported but unused. Remove?"                                 |
| `.first`/`.last` on Set   | Collection type      | "Calling `.first` on unordered Set. Make [field] `List<T>`?"                  |
| Custom dot-method         | Syntax error         | "Use free-standing: `filter(collection, pred)` not `collection.filter(pred)`" |
| Missing version header    | Format error         | "Add `-- allium: 3` as first line."                                           |

---

## Priority Rules

| Situation                | Action                                          |
| ------------------------ | ----------------------------------------------- |
| `check` errors present   | Block writes; fix errors first                  |
| `check` warnings present | Show human; need explicit decision before write |
| `check` exits 0          | Proceed to `analyse`                            |

---

## Auto-Fix vs. Surface to Human

**Auto-fix (no approval):**
- Missing version header (`-- allium: 3`) — add silently
- Unused `use` imports — remove silently
- Formatting issues (indentation, spacing)

**Always surface:**
- Any `error`/`warning` on behavioral semantics
- `data_flow` findings (human knows if dead data intentional)
- Stale `traces:` references (human knows current location)

---

## After Fixes

Re-run after fixes, confirm no regressions:

```bash
allium check <file>    # must exit 0
```
