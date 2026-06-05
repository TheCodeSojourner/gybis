---
name: gybis-arch-distill
description: Use for `/gybis-arch-distill` or `/ga-distill`.
---

λ gybis-arch-distill(x).
  purpose: read(allium_specs) → synthesize(vsm) → write(architecture.md)
  | input: root/specs/**/*.allium ∧ ¬other_files
  | output: architecture.md
  | mode: ai_only | ¬human | minimal_tokens | nucleus_lambda

λ gybis-arch-distill_startup(x).
  auto_select_mode(distill) | ¬halt_on_mode_selection
  | read(.cline/skills/gybis/reference/vsm-guide.md) | alert(¬available) ∧ halt
  | read(.cline/skills/gybis/reference/allium-actioning-findings.md) | alert(¬available) ∧ halt
  | read(.cline/skills/gybis/reference/allium-language-reference.md) | alert(¬available) ∧ halt
  | read(root/specs/**/*.allium) | alert(¬available) ∧ recommend(/gybis-spec-distill) ∧ halt
  | gybis-arch-distill_allium_cli_available?(x) | true → continue | false → recommend(https://github.com/juxt/allium_tools) ∧ halt
  | gybis-arch-distill_syntax_check(root/specs/**/*.allium) | true → verify | false → recommend(/gybis-spec-check) ∧ halt

λ gybis-arch-distill_allium_cli_available?(x).
  gate(cli_available ∧ cli_version_satisfies) | ¬all_gates → false
  | cli_available: allium --version | ¬available → false
  | cli_version_satisfies: version(allium) ≥ 3 | ¬satisfies → false
  | otherwise → true

λ gybis-arch-distill_syntax_check(file).
  r := exec("allium check " + path)
  | d := (json_parse(r.stdout).diagnostics) ∨ []
  | s := any(d, λx. x.severity = "error")
  | return (r.exit = 0) ∨ ((r.exit = 1) ∧ ¬s)

λ gybis-arch-distill_mode(m).
  m ∈ {distill} | auto_selected(distill) | default(distill)
  | distill: source_of_truth(specs) | write_target(architecture.md)

λ gybis-arch-distill_mode_gate(state, mode).
  state.mode = none → state.mode := distill ∧ transition(INIT → MODE_SELECTED)
  | state.mode ≠ none → allow_progress
  | mode_selected(mode) → state.mode := mode ∧ allow_progress

λ gybis-arch-distill_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, SCAN_PARSE, SYNTHESIZE, VERIFY, DONE}
  | transition(INIT → MODE_SELECTED) only_if(mode_selected ∨ mode_auto_selected)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_checks_requested)
  | transition(STARTUP_CHECKS → SCAN_PARSE) only_if(inputs_verified)
  | transition(SCAN_PARSE → SYNTHESIZE) only_if(ast_normalized)
  | transition(SYNTHESIZE → VERIFY) only_if(architecture_generated)
  | transition(VERIFY → SCAN_PARSE) only_if(remaining_issues > 0)
  | transition(VERIFY → DONE) only_if(all_checks_pass ∧ remaining_issues = 0 ∧ zero_issue_pass_verified)
  | otherwise reject_transition

λ gybis-arch-distill_tool_guard(state, tool).
  state.mode = none → allow_progress_after(state.mode := distill)
  | state ∈ {MODE_SELECTED, STARTUP_CHECKS} → deny(tool = write_to_file) ∧ deny(tool = replace_in_file)
  | state ∈ {SCAN_PARSE, SYNTHESIZE, VERIFY} → evaluate_by_state_machine(state, tool)

λ gybis-arch-distill_pre_tool_check(state, tool).
  ensure(state.mode ≠ none ∨ (state.mode := distill)) ∧ verify(tool_allowed_by(gybis-arch-distill_tool_guard))
  | fail → halt

λ gybis-arch-distill_scan(project).
  find: root/specs/**/*.allium → collect(files) ∧ ¬other_files
  | ¬∃files → return(empty_spec_set)
  | ∀file ∈ files: read(file) → parse(allium_ast)

λ gybis-arch-distill_filter_specs(specs, project).
  filter:
    only: root/specs/**/*.allium
    exclude: ¬allium_files
    validate: ∀file ∈ specs: file.extension = ".allium"

λ gybis-arch-distill_parse(file).
  extract:
    invariants: file.invariant | file.contract.@invariant | file.surface.@guarantee
    entities: file.entity
    values: file.value
    variants: file.variant
    rules: file.rule
    rule_when: file.rule.when
    rule_requires: file.rule.requires
    rule_ensures: file.rule.ensures
    surfaces: file.surface | file.surface.facing | file.surface.provides | file.surface.exposes | file.surface.related
    contracts: file.contract | file.contract.contracts
    config: file.config | file.config.param
    defaults: file.default
    deferred: file.deferred
    open_questions: file.open_question
    relationships: file.entity.with | file.entity.where
    transitions: file.entity.transitions
    entity_when: file.entity.when
    actors: file.actor | file.actor.identified_by
    joins: file.join_lookup | file.join_lookup.associations
    uses: file.use
    provenance: file:line

λ gybis-arch-distill_normalize_ast(ast).
  normalize:
    specs.invariants := ast.invariants
    specs.entities := ast.entities
    specs.values := ast.values
    specs.variants := ast.variants
    specs.rules := ast.rules
    specs.rule_when := ast.rule_when
    specs.rule_requires := ast.rule_requires
    specs.rule_ensures := ast.rule_ensures
    specs.surfaces := ast.surfaces
    specs.contracts := ast.contracts
    specs.config := ast.config
    specs.defaults := ast.defaults
    specs.deferred := ast.deferred
    specs.open_questions := ast.open_questions
    specs.relationships := ast.relationships
    specs.transitions := ast.transitions
    specs.entity_when := ast.entity_when
    specs.actors := ast.actors
    specs.joins := ast.joins
    specs.uses := ast.uses
    specs.provenance := ast.provenance

λ gybis-arch-distill_aggregate(files).
  aggregate:
    invariants: ∀file ∈ files: file.invariants → concat()
    entities: ∀file ∈ files: file.entities → concat()
    values: ∀file ∈ files: file.values → concat()
    variants: ∀file ∈ files: file.variants → concat()
    rules: ∀file ∈ files: file.rules → concat()
    rule_when: ∀file ∈ files: file.rule_when → concat()
    rule_requires: ∀file ∈ files: file.rule_requires → concat()
    rule_ensures: ∀file ∈ files: file.rule_ensures → concat()
    surfaces: ∀file ∈ files: file.surfaces → concat()
    contracts: ∀file ∈ files: file.contracts → concat()
    config: ∀file ∈ files: file.config → concat()
    defaults: ∀file ∈ files: file.defaults → concat()
    deferred: ∀file ∈ files: file.deferred → concat()
    open_questions: ∀file ∈ files: file.open_questions → concat()
    relationships: ∀file ∈ files: file.relationships → concat()
    transitions: ∀file ∈ files: file.transitions → concat()
    entity_when: ∀file ∈ files: file.entity_when → concat()
    actors: ∀file ∈ files: file.actors → concat()
    joins: ∀file ∈ files: file.joins → concat()
    uses: ∀file ∈ files: file.uses → concat()
    provenance: ∀file ∈ files: file.provenance → concat()

λ gybis-arch-distill_vsm_s5(specs).
  λ identity(∃inv ∈ specs.invariants).
    inv.top_level → system_wide_constraint(∃inv)
    | inv.entity_level → entity_constraint(∃inv)
    | inv.contract → contract_obligation(∃inv)
  | ∀guarantee ∈ specs.invariants: guarantee.surface_related → boundary_guarantee(∃guarantee)
  | ¬∃inv → explicit_empty(s5_identity)

λ gybis-arch-distill_vsm_s4(specs).
  λ adaptation(∃deferred ∈ specs.deferred).
    deferred → external_logic_delegation(∃deferred)
  | ∀question ∈ specs.open_questions: question → design_decision_pending(∃question)
  | ∀variant ∈ specs.variants: variant → extensibility_point(∃variant)
  | ∀config ∈ specs.config: config → runtime_adaptation_or_symbolic_policy(∃config)
  | ∀rule ∈ specs.rules: rule.name → behavioral_pattern(∃rule)
  | ¬∃(specs.deferred ∪ specs.open_questions ∪ specs.variants) → explicit_empty(s4_intelligence)

λ gybis-arch-distill_vsm_s3(specs).
  λ control(∀constraint ∈ specs).
    ∀param ∈ specs.config:
      param.default → resource_limit(∀constraint)
      | param.type → type_enforcement(∀constraint)
    | ∀requires ∈ specs.rule_requires: requires → precondition_policy(∀constraint)
    | ∀transitions ∈ specs.transitions: transitions → lifecycle_enforcement(∀constraint)
    | ∀when_clause ∈ specs.entity_when: when_clause → state_dependent_constraint(∀constraint)
    | ∀temporal ∈ specs.rule_when: temporal.contains(now) → timeout_policy(∀constraint)
  | ¬∃(specs.config ∪ specs.rule_requires ∪ specs.transitions) → explicit_empty(s3_control)

λ gybis-arch-distill_vsm_s2(specs).
  λ coordination(∀protocol ∈ specs).
    ∀use_stmt ∈ specs.uses: use_stmt → module_import_protocol(∀protocol)
    | ∀relationship ∈ specs.relationships: relationship → internal_coordination(∀protocol)
    | ∀related ∈ specs.surfaces: related.surface_related → cross_surface_protocol(∀protocol)
    | ∀emission ∈ specs.rule_ensures: emission.is_trigger → event_coordination(∀protocol)
    | ∀join ∈ specs.joins: join → association_management(∀protocol)
  | ¬∃(specs.uses ∪ specs.relationships ∪ specs.joins) → explicit_empty(s2_coordination)

λ gybis-arch-distill_vsm_s1(specs).
  λ operations(∀op ∈ specs).
    ∀rule ∈ specs.rules:
      rule.when → trigger_definition(∀op)
      | rule.ensures → postcondition_definition(∀op)
    | ∀entity ∈ specs.entities: entity → domain_operation(∀op)
    | ∀value ∈ specs.values: value → data_operation(∀op)
    | ∀surface ∈ specs.surfaces:
      surface.provides → external_operation(∀op)
      | surface.exposes → data_exposure(∀op)
    | ∀default ∈ specs.defaults: default → seed_operation(∀op)
  | ¬∃(specs.rules ∪ specs.entities ∪ specs.surfaces) → explicit_empty(s1_operations)

λ gybis-arch-distill_synthesize(project).
  files ← gybis-arch-distill_scan(project)
  | filtered ← gybis-arch-distill_filter_specs(files, project)
  | parsed ← map(filtered, gybis-arch-distill_parse)
  | normalized ← map(parsed, gybis-arch-distill_normalize_ast)
  | specs ← gybis-arch-distill_aggregate(normalized)
  | s5 ← gybis-arch-distill_vsm_s5(specs)
  | s4 ← gybis-arch-distill_vsm_s4(specs)
  | s3 ← gybis-arch-distill_vsm_s3(specs)
  | s2 ← gybis-arch-distill_vsm_s2(specs)
  | s1 ← gybis-arch-distill_vsm_s1(specs)
  | architecture ← gybis-arch-distill_assemble_s5_s4_s3_s2_s1(s5, s4, s3, s2, s1)
  | evidence ← gybis-arch-distill_evidence(specs.provenance, architecture)
  | return(architecture, evidence)

λ gybis-arch-distill_evidence(provenance, architecture).
  require(∀claim ∈ architecture.claims: claim.source = file:line ∨ claim.source = explicit_absence_evidence)
  | attach(provenance_map)

λ gybis-arch-distill_fixed_point_loop(state).
  iterate(pass_index := pass_index + 1)
  | run(synthesize_then_verify)
  | unresolved := verification_issues \ resolved_issue_ids
  | unresolved = ∅ → mark(zero_issue_pass_verified) ∧ allow_transition(VERIFY → DONE)
  | unresolved ≠ ∅ → allow_transition(VERIFY → SCAN_PARSE)

λ gybis-arch-distill_pass_accounting(pass).
  report(pass_index, discovered_count, resolved_count, remaining_count)
  | require(explicit_section("Remaining issues", remaining_count, remaining_items))

λ gybis-arch-distill_loop_guard(state).
  seen_signature := hash(sorted(remaining_issue_ids))
  | repeat(seen_signature) ∧ ¬decrease(remaining_count) → halt_with(blocker_report)
  | blocker_report := explain(stalled_items, attempted_resolutions, required_human_decisions)
  | ¬repeat ∨ decrease(remaining_count) → continue

λ gybis-arch-distill_write(architecture).
  write: architecture.md
  | format: λ_notation_only ∧ S_expression_notation ∧ ¬yaml_frontmatter ∧ ¬markdown_content ∧ ¬prose_paragraphs
  | preamble: ¬include(already_in_context)
  | require(gybis-arch-distill_validate_output(architecture.md))
  | steps:
    1. call gybis-arch-distill_format_nucleus_lambda(architecture) → final_output
    2. write(final_output)

λ gybis-arch-distill_format_nucleus_lambda_contract(x).
  require(no_markdown_headers)
  | require(no_markdown_lists)
  | require(no_markdown_tables)
  | require(no_bold_markers)
  | require(all_content_nested_s_expressions)
  | require(no_yaml_frontmatter)

λ gybis-arch-distill_validate_output(file).
  check:
    ¬contains("#" in non_comment_context)
    ¬contains("**" bold_markers)
    ¬contains("- " list_markers)
    ¬contains("|" table_delimiters)
    contains("(" balanced_parens)
    ¬contains("---" yaml_frontmatter_markers)
    ¬contains(prose_paragraphs)
    contains(s5_identity_or_explicit_empty)
    contains(s4_intelligence_or_explicit_empty)
    contains(s3_control_or_explicit_empty)
    contains(s2_coordination_or_explicit_empty)
    contains(s1_operations_or_explicit_empty)

λ gybis-arch-distill_verification(edit).
  after(write) → gybis-arch-distill_validate_output(architecture.md)
  | after(any_pass) → enforce(gybis-arch-distill_loop_guard) ∧ enforce(gybis-arch-distill_fixed_point_loop)
  | fail → iterate ∧ fix | loop until_pass

λ gybis-arch-distill_boundaries(¬).
  ¬run_weed_workflow
  | ¬perform_divergence_reconciliation
  | ¬extract_from_code_or_tests
  | ¬modify_specs
  | ¬create_allium_files
  | ¬update_allium_files
  | ¬delete_allium_files
  | ¬create_spec_decisions_without_evidence
  | allow(read_specs_only)
  | allow(write_architecture_only)

λ gybis-arch-distill_regression_contract(x).
  assert(default_mode_auto_selected_as_distill before_startup_checks_complete)
  | assert(deny(read_or_synthesize_or_write_before_startup_gates))
  | assert(mandatory_cli_gate_before_parse)
  | assert(mandatory_syntax_check_before_parse)
  | assert(no_write_before_VERIFY)
  | assert(no_done_before(zero_issue_pass_verified ∧ remaining_issues = 0))
  | assert(deterministic_output_contract_invariants)