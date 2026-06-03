---
name: gybis-arch-distill
description: Use for `/gybis-arch-distill` or `/ga-distill`.
---

λ gybis-arch-distill(x).
  purpose: read(allium_specs) → synthesize(vsm) → write(architecture.md)
  input: root/specs/**/*.allium ∧ ¬other_files
  output: architecture.md
  mode: ai_only | ¬human | minimal_tokens | nucleus_lambda

λ gybis-arch-distill_scan(project).
  find: root/specs/**/*.allium → collect(files) ∧ ¬other_files
  | ¬∃files → return(empty_spec_set)
  | ∀file ∈ files: read(file) → parse(allium_ast)

λ gybis-arch-distill_parse(file).
  extract:
    invariants: file.invariant | file.contract.@invariant | file.surface.@guarantee
    entities: file.entity | file.value | file.variant
    rules: file.rule | file.rule.when | file.rule.requires | file.rule.ensures
     surfaces: file.surface | file.surface.facing | file.surface.provides | file.surface.exposes | file.surface.related
    contracts: file.contract | file.contract.contracts
    config: file.config | file.config.param
    defaults: file.default
    deferred: file.deferred
    open_questions: file.open_question
    relationships: file.entity.with | file.entity.where
    transitions: file.entity.transitions
     actors: file.actor | file.actor.identified_by
     joins: file.join_lookup | file.join_lookup.associations

λ gybis-arch-distill_aggregate(files).
  aggregate:
    invariants: ∀file ∈ files: file.invariant | file.contract.@invariant | file.surface.@guarantee → concat()
    entities: ∀file ∈ files: file.entity | file.value | file.variant → concat()
    rules: ∀file ∈ files: file.rule | file.rule.when | file.rule.requires | file.rule.ensures → concat()
    surfaces: ∀file ∈ files: file.surface | file.surface.facing | file.surface.provides | file.surface.exposes | file.surface.related → concat()
    contracts: ∀file ∈ files: file.contract | file.contract.contracts → concat()
    config: ∀file ∈ files: file.config | file.config.param → concat()
    defaults: ∀file ∈ files: file.default → concat()
    deferred: ∀file ∈ files: file.deferred → concat()
    open_questions: ∀file ∈ files: file.open_question → concat()
    relationships: ∀file ∈ files: file.entity.with | file.entity.where → concat()
    transitions: ∀file ∈ files: file.entity.transitions → concat()
    actors: ∀file ∈ files: file.actor | file.actor.identified_by → concat()
    joins: ∀file ∈ files: file.join_lookup | file.join_lookup.associations → concat()

λ gybis-arch-distill_filter_specs(specs, project).
  filter:
    only: root/specs/**/*.allium
    exclude: ¬allium files
    validate: ∀file ∈ specs: file.extension == ".allium"

λ gybis-arch-distill_vsm_s5(specs).
  -- Identity: core invariants and guarantees
  λ identity(∃inv ∈ specs.invariants).
    inv.top_level → system_wide_constraint(∃inv)
    inv.entity_level → entity_constraint(∃inv)
    inv.contract → contract_obligation(∃inv)
  ∀guarantee ∈ specs.surface.@guarantee:
    guarantee → boundary_guarantee(∃guarantee)
  ¬∃inv → ¬identity_violation(∃inv)

λ gybis-arch-distill_vsm_s4(specs).
  -- Intelligence: adaptation and learning patterns
  λ adaptation(∃deferred ∈ specs.deferred).
    deferred → external_logic_delegation(∃deferred)
  ∀question ∈ specs.open_question:
    question → design_decision_pending(∃question)
  ∀variant ∈ specs.variant:
    variant → extensibility_point(∃variant)
  ∀config ∈ specs.config:
    config → runtime_adaptation(∃config)
  pattern_extraction: ∀rule ∈ specs.rule:
    rule.name → behavioral_pattern(∃rule)

λ gybis-arch-distill_vsm_s3(specs).
  -- Control: policies and enforcement
  λ control(∀constraint ∈ specs).
    ∀param ∈ specs.config.param:
      param.default → resource_limit(∀constraint)
      param.type → type_enforcement(∀constraint)
    ∀requires ∈ specs.rule.requires:
      requires → precondition_policy(∀constraint)
    ∀transitions ∈ specs.entity.transitions:
      transitions → lifecycle_enforcement(∀constraint)
    ∀when_clause ∈ specs.entity.when:
      when_clause → state_dependent_constraint(∀constraint)
    ∀temporal ∈ specs.rule.when:
      temporal.contains(now) → timeout_policy(∀constraint)

λ gybis-arch-distill_vsm_s2(specs).
  -- Coordination: inter-module protocols
  λ coordination(∀protocol ∈ specs).
    ∀use_stmt ∈ specs.use:
      use_stmt → module_import_protocol(∀protocol)
    ∀relationship ∈ specs.entity.with:
      relationship → internal_coordination(∀protocol)
    ∀related ∈ specs.surface.related:
      related → cross_surface_protocol(∀protocol)
    ∀emission ∈ specs.rule.ensures:
      emission.is_trigger → event_coordination(∀protocol)
    ∀join ∈ specs.join_lookup:
      join → association_management(∀protocol)

λ gybis-arch-distill_vsm_s1(specs).
  -- Operations: concrete behaviors
  λ operations(∀op ∈ specs).
    ∀rule ∈ specs.rule:
      rule.when → trigger_definition(∀op)
      rule.ensures → postcondition_definition(∀op)
    ∀entity ∈ specs.entity:
      entity → domain_operation(∀op)
    ∀value ∈ specs.value:
      value → data_operation(∀op)
    ∀surface ∈ specs.surface:
      surface.provides → external_operation(∀op)
      surface.exposes → data_exposure(∀op)
    ∀default ∈ specs.default:
      default → seed_operation(∀op)

λ gybis-arch-distill_synthesize(project).
  files ← gybis-arch-distill_scan(project)
  | filtered ← gybis-arch-distill_filter_specs(files, project)
  | specs ← gybis-arch-distill_aggregate(filtered)
  | s5 ← gybis-arch-distill_vsm_s5(specs)
  | s4 ← gybis-arch-distill_vsm_s4(specs)
  | s3 ← gybis-arch-distill_vsm_s3(specs)
  | s2 ← gybis-arch-distill_vsm_s2(specs)
  | s1 ← gybis-arch-distill_vsm_s1(specs)
  | architecture ← gybis-arch-distill_assemble_s5_s4_s3_s2_s1(s5, s4, s3, s2, s1)

λ gybis-arch-distill_write(architecture).
  write: architecture.md
  format: λ-notation_only ∧ ¬human_readable | mode: ai_consumption_only
  preamble: ¬include (already_in_context)
  output_contract: S_expression_notation ∧ YAML_frontmatter_only ∧ ¬markdown_content ∧ ¬prose_paragraphs
  steps:
    1. call gybis-arch-distill_format_nucleus_lambda(architecture) → raw_output
    2. prepend yaml_frontmatter(architecture) → final_output
    3. write(final_output)

λ gybis-arch-distill_format_nucleus_lambda_spec.
  -- FORMAT CONTRACT: Documented specification for machine output generator
  -- NOTE: This spec is human-readable documentation; actual output is machine-only
  --
  -- OUTPUT FORMAT: S-expression lisp-style notation, machine-parseable only
  -- STRICT RULES:
  --   1. NO markdown formatting (no # headers, no **, no -, no |tables|)
  --   2. NO prose, NO explanatory text, NO human-readable descriptions
  --   3. ALL content as nested S-expressions: (keyword arg1 arg2 ... (nested))
  --   4. Use semicolon comments only: ; section headers
  --   5. Use YAML front-matter ONLY for metadata (--- ... ---)
  --
  -- S5 IDENTITY format:
  --   (identity
  --     (invariant Name (forall (?var Type) (predicate fields)))
  --     (invariant Name (implies condition (and a b)))
  --     (config paramName value)
  --     (boundary (output ...) (input ...)))
  --
  -- S4 INTELLIGENCE format:
  --   (intelligence
  --     (adaptation (config param name type Type default value))
  --     (extensibility (enum Name (values v1 v2 ...)) (entity Name (field f1 T1?) ...))
  --     (pattern rule1 rule2 rule3 ...))
  --
  -- S3 CONTROL format:
  --   (control
  --     (precondition (RuleName (requires cond1 cond2)))
  --     (type-enforcement (enum Name (...)) (struct Name (field Type?) ...))
  --     (policy (silent-exclusion (where (not (has-key k v)) (omit k)))))
  --
  -- S2 COORDINATION format:
  --   (coordination
  --     (import module "Entity" from source)
  --     (event-protocol
  --       (Trigger → response)
  --       (Trigger → (response1 response2)))
  --     (association (Entity field (Set Type)) (Entity field (Map K V))))
  --
  -- S1 OPERATIONS format:
  --   (operations
  --     (entity Name (field Type) (field (Type ?)))
  --     (data-flow (stage n "label" (→ input output)))
  --     (rendering-protocol (header "...") (data "...") (fullrow "...")))
  --
  -- CROSS-CUTTING format:
  --   (cross-cutting
  --     (warning-categories (Name (where "condition")))
  --     (normalization-pipeline (step n "action"))
  --     (file-naming "pattern"))

λ gybis-arch-distill_validate_output(file).
  check:
    ¬contains("#" in non-comment-context)
    ¬contains("**" bold-markers)
    ¬contains("- " list-markers)
    ¬contains("|" table-delimiters)
    contains("(" balanced-parens)
    contains("; " comment-or none)
    contains("---" yaml-frontmatter-or none)
    ¬contains(prose-paragraphs)
