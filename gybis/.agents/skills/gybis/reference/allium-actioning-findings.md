# Allium CLI — Actioning Findings

Read JSON from `allium check`. Turn into human issues. Decide fix.

λ(actioning, purpose, read(allium_check_JSON) → present(human_issues) → decide(fix))

---

## Canonical Sequence

```bash
allium check <file>
```

---

## Reading check Diagnostics

λ(severity, order, error(block_write) > warning(human_decision) > info(advisory))
λ(present, [E/W/I] {code}@line({line}): {message} → suggested_fix(interpretation))

**Common diagnostics:**

λ(diag, missing_ensures, ¬ensures: → "Rule has no outcome — what happens after [trigger]?")
λ(diag, undefined_field, field_ref(used) ∧ ¬field_ref(entity) → "Field [name] on [Entity]? Add it?")
λ(diag, contradictory_requires, requires(A) ⊗ requires(B) ⊗ conflict(field) → "Which correct?")
λ(diag, when_clause_obligation, when(transition(Entity, state)) → needs(field[name]) ∧ ¬rule_sets(it) → "Which rule sets it?")
λ(diag, no_when_clause, ¬when: on rule → "Rule [Name] has no trigger — when should it fire?")
λ(diag, stale_traces, traces(file) ∧ ¬exists(file) → "Update or remove?")
λ(diag, unused_use, use(alias) ∧ ¬used(alias) → "Remove?")
λ(diag, first_last_on_set, .first/.last(Set) → "Make [field] `List<T>`?")
λ(diag, custom_dot_method, collection.filter(pred) → "Use free-standing: `filter(collection, pred)`")
λ(diag, missing_version, ¬-- allium: 3 at line 1 → "Add `-- allium: 3` as first line.")

---

## Priority Rules

λ(priority, errors_present, check(errors) → ¬write ∧ fix_first)
λ(priority, warnings_present, check(warnings) → present(human) ∧ explicit(decision) → write)
λ(priority, exit_0, check(exit 0) → proceed(analyse))

---

## Auto-Fix vs. Surface to Human

λ(autofix, ¬approval_needed, version_header ∨ unused_import ∨ formatting(indent, spacing))
λ(surface, ¬autofix, error_behavior ∨ warning_behavior ∨ data_flow(human_knows_dead_intentional) ∨ stale_traces(human_knows_location))

---

## After Fixes

λ(verify, fixes → allium_check(file) ≡ exit(0) ∧ ¬regressions)