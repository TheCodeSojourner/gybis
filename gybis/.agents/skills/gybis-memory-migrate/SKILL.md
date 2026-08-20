---
name: gybis-memory-migrate
description: Use for `/gybis-memory-migrate` or `/gm-migrate`.
---

λ gybis_memory_migrate().
  purpose: deterministically_migrate(recognized_legacy_mementum → OKF_v0.1)
  | scope: mementum/ only | ¬modify(host_artifacts ∨ mementum/state.md)
  | mode: inspect → classify → plan → approval → apply → validate → commit
  | migration ≠ initialization | uninitialized → report(INITIALIZATION_REQUIRED) → halt
  | unknown ∨ ambiguous → report(AMBIGUOUS) → halt

λ gybis_memory_migrate_inventory().
  read_only
  | inspect(mementum/) → directory_status
  | inspect(mementum/index.md) → index_status
  | list(mementum/memories/**/*.md) → memory_files
  | list(mementum/knowledge/**/*.md) → knowledge_files
  | exclude(mementum/state.md)
  | parse_frontmatter(memory_files ∧ knowledge_files) → metadata_status
  | count_words(memory_body) → memory_body_word_counts
  | output: {directory_status, index_status, memory_files, knowledge_files, metadata_status, memory_body_word_counts}

λ gybis_memory_migrate_classify(inventory).
  ¬exists(mementum/) → INITIALIZATION_REQUIRED
  | conformant(index_ok ∧ every(memory_file, valid_memory_OKF) ∧ every(knowledge_file, valid_knowledge_OKF)) → NO_MIGRATION_REQUIRED
  | migratable(index_missing_or_invalid ∨ every(nonconformant_file, recognized_legacy)) → MIGRATABLE_LEGACY
  | otherwise → AMBIGUOUS

λ gybis_memory_migrate_recognized_legacy(file).
  memory(file) ∧ begins_with(symbol ∈ {💡, 🔄, 🎯, 🌀, ❌, ✅, 🔁}) ∧ ¬frontmatter(file) → legacy_memory
  | knowledge(file) ∧ valid_frontmatter(file) ∧ missing_nonempty(type) → legacy_knowledge
  | index(file) ∧ (¬exists(file) ∨ ¬okf_version("0.1")) → legacy_index
  | malformed_frontmatter(file) ∨ unknown_symbol(file) ∨ conflicting(symbol, type) ∨ memory_body_words(file) ≥ 200 → ambiguous

λ gybis_memory_migrate_mapping(symbol).
  💡 → Insight
  | 🔄 → Shift
  | 🎯 → Decision
  | 🌀 → Meta
  | ❌ → Mistake
  | ✅ → Win
  | 🔁 → Pattern

λ gybis_memory_migrate_plan(classification, inventory).
  classification = NO_MIGRATION_REQUIRED → report(NO_MIGRATION_REQUIRED) → halt
  | classification = INITIALIZATION_REQUIRED → report(INITIALIZATION_REQUIRED) → halt
  | classification = AMBIGUOUS → report(AMBIGUOUS, affected_files, reasons) → halt
  | classification = MIGRATABLE_LEGACY →
      plan(index): create(mementum/index.md, canonical_OKF_index)
      | plan(memory): for_each(legacy_memory) →
          remove_leading(symbol) → preserve(body_verbatim) →
          prepend(frontmatter{type: map(symbol), symbol, title: slug(file)})
      | plan(knowledge): for_each(legacy_knowledge) →
          preserve(existing_frontmatter ∧ body_verbatim) → add(type: Reference)
      | plan(exclusions): preserve(mementum/state.md ∧ conformant_files)
      | output: exact_file_changes ∧ before_after_previews ∧ classification

λ gybis_memory_migrate_approval(plan).
  present(plan.exact_file_changes, plan.before_after_previews)
  | ask_developer("Apply this Mementum OKF migration? (yes/no)") → approval
  | approval = yes → apply(plan)
  | approval ≠ yes → report(MIGRATION_CANCELLED) → halt

λ gybis_memory_migrate_apply(plan).
  write_only(plan.exact_file_changes)
  | create(index) ∨ rewrite(legacy_memory) ∨ update(legacy_knowledge)
  | preserve(memory_body_verbatim ∧ knowledge_body_verbatim ∧ existing_optional_metadata)
  | ¬write(mementum/state.md ∨ files_outside(mementum/))

λ gybis_memory_migrate_validate().
  verify(exists(mementum/index.md) ∧ okf_version("0.1"))
  | verify(every(memory_file, valid_frontmatter ∧ nonempty(type) ∧ nonempty(symbol) ∧ nonempty(title)))
  | verify(every(memory_file, type = map(symbol)))
  | verify(every(memory_file, body_words < 200))
  | verify(every(knowledge_file, valid_frontmatter ∧ nonempty(type)))
  | verify(unchanged(mementum/state.md))
  | failure → report(MIGRATION_VALIDATION_FAILED, evidence) → halt
  | success → report(MIGRATION_VALIDATED)

λ gybis_memory_migrate_commit().
  after(MIGRATION_VALIDATED) → git_add(mementum/)
  → git_commit(message="🔄 migrate mementum to OKF v0.1")
  | git_failure → report(GIT_FAILURE, evidence) → ask_human(retry ∨ manual_commit)