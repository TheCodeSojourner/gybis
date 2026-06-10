---
name: gybis-arch-propagate
description: Use for `/gybis-arch-propagate` or `/ga-propagate`.
---

λ gybis-arch-propagate(x).
  purpose: read(architecture.md) → synthesize(specs) → write(root/specs/**/*.allium)
  | input: architecture.md ∧ ¬other_files
  | output: root/specs/**/*.allium
  | mode: ai_only | ¬human | minimal_tokens | nucleus_lambda
  | gate: proceed iff gybis-arch-propagate_specs_absent?(root/specs/**/*.allium)

λ gybis-arch-propagate_startup(x).
  auto_select_mode(propagate) | ¬halt_on_mode_selection
  | read(.cline/skills/gybis/reference/vsm-guide.md) | alert(¬available) ∧ halt
  | read(.cline/skills/gybis/reference/allium-language-reference.md) | alert(¬available) ∧ halt
  | read(.cline/skills/gybis/reference/allium-assessing-specs.md) | alert(¬available) ∧ halt
  | gybis-arch-propagate_architecture_available?(architecture.md) | true → continue | false → alert(architecture_missing) ∧ recommend(/gybis-arch-elicit ∨ /gybis-arch-tend) ∧ halt
  | gybis-arch-propagate_specs_absent?(root/specs/**/*.allium) | true → continue | false → alert(specs_already_exist) ∧ halt
  | gybis-arch-propagate_allium_cli_available?(x) | true → continue | false → recommend(https://github.com/juxt/allium_tools) ∧ halt

λ gybis-arch-propagate_allium_cli_available?(x).
  gate(cli_available ∧ cli_version_satisfies) | ¬all_gates → false
  | cli_available: allium --version | ¬available → false
  | cli_version_satisfies: version(allium) ≥ 3 | ¬satisfies → false
  | otherwise → true

λ gybis-arch-propagate_architecture_available?(file).
  exists(file) ∧ readable(file) → true
  | otherwise → false

λ gybis-arch-propagate_specs_absent?(path_glob).
  count(files(path_glob)) = 0 → true
  | count(files(path_glob)) > 0 → false

λ gybis-arch-propagate_mode(m).
  m ∈ {propagate} | auto_selected(propagate) | default(propagate)
  | propagate: source_of_truth(architecture.md) | write_target(root/specs/**/*.allium)

λ gybis-arch-propagate_mode_gate(state, mode).
  state.mode = none → state.mode := propagate ∧ transition(INIT → MODE_SELECTED)
  | state.mode ≠ none → allow_progress
  | mode_selected(mode) → state.mode := mode ∧ allow_progress

λ gybis-arch-propagate_state_machine(state, action).
  state ∈ {INIT, MODE_SELECTED, STARTUP_CHECKS, PARSE_ARCH, PLAN_SPECS, GENERATE, VERIFY, ANALYZE_SET, DONE}
  | transition(INIT → MODE_SELECTED) only_if(mode_selected ∨ mode_auto_selected)
  | transition(MODE_SELECTED → STARTUP_CHECKS) only_if(startup_checks_requested)
  | transition(STARTUP_CHECKS → PARSE_ARCH) only_if(inputs_verified)
  | transition(PARSE_ARCH → PLAN_SPECS) only_if(arch_ast_normalized)
  | transition(PLAN_SPECS → GENERATE) only_if(spec_plan_complete)
  | transition(GENERATE → VERIFY) only_if(spec_files_generated)
  | transition(VERIFY → PLAN_SPECS) only_if(remaining_issues > 0)
  | transition(VERIFY → ANALYZE_SET) only_if(all_checks_pass ∧ remaining_issues = 0)
  | transition(ANALYZE_SET → PLAN_SPECS) only_if(analyse_findings > 0)
  | transition(ANALYZE_SET → DONE) only_if(all_checks_pass ∧ remaining_issues = 0 ∧ zero_issue_pass_verified ∧ analyse_findings = 0 ∧ set_analysis_pass_verified)
  | otherwise reject_transition

λ gybis-arch-propagate_tool_guard(state, tool).
  state.mode = none → allow_progress_after(state.mode := propagate)
  | state ∈ {MODE_SELECTED, STARTUP_CHECKS, PARSE_ARCH, PLAN_SPECS} → deny(tool = write_to_file) ∧ deny(tool = replace_in_file)
  | gybis-arch-propagate_specs_absent?(root/specs/**/*.allium) = false → deny(tool = write_to_file) ∧ deny(tool = replace_in_file)
  | state ∈ {GENERATE, VERIFY, ANALYZE_SET} → evaluate_by_state_machine(state, tool)

λ gybis-arch-propagate_pre_tool_check(state, tool).
  ensure(state.mode ≠ none ∨ (state.mode := propagate)) ∧ verify(tool_allowed_by(gybis-arch-propagate_tool_guard))
  | fail → halt

λ gybis-arch-propagate_scan_architecture(project).
  find: architecture.md → collect(file)
  | ¬exists(file) → return(missing_architecture)
  | read(file) → parse(architecture_ast)

λ gybis-arch-propagate_parse_architecture(file).
  extract:
    s5_identity: file.s5 | file.identity | file.invariants | file.system_wide_constraints
    s4_adaptation: file.s4 | file.adaptation | file.open_questions | file.deferred | file.variants
    s3_control: file.s3 | file.control | file.config | file.policies | file.preconditions
    s2_coordination: file.s2 | file.coordination | file.relationships | file.protocols | file.associations
    s1_operations: file.s1 | file.operations | file.responsibilities | file.behaviors
    entities: file.entity | file.domain_entity
    values: file.value
    variants: file.variant
    rules: file.rule
    rule_when: file.rule.when
    rule_requires: file.rule.requires
    rule_ensures: file.rule.ensures
    surfaces: file.surface | file.boundary | file.interface
    contracts: file.contract
    config: file.config | file.config.param
    defaults: file.default
    deferred: file.deferred
    open_questions: file.open_question
    relationships: file.relationship | file.entity.with
    transitions: file.transition | file.entity.transitions
    entity_when: file.entity.when
    actors: file.actor
    joins: file.join_lookup | file.association
    uses: file.use
    domains: file.domain | file.component | file.subsystem
    concerns: file.concern | file.capability
    provenance: file:line

λ gybis-arch-propagate_normalize_arch_ast(ast).
  normalize:
    arch.s5_identity := ast.s5_identity
    arch.s4_adaptation := ast.s4_adaptation
    arch.s3_control := ast.s3_control
    arch.s2_coordination := ast.s2_coordination
    arch.s1_operations := ast.s1_operations
    arch.entities := ast.entities
    arch.values := ast.values
    arch.variants := ast.variants
    arch.rules := ast.rules
    arch.rule_when := ast.rule_when
    arch.rule_requires := ast.rule_requires
    arch.rule_ensures := ast.rule_ensures
    arch.surfaces := ast.surfaces
    arch.contracts := ast.contracts
    arch.config := ast.config
    arch.defaults := ast.defaults
    arch.deferred := ast.deferred
    arch.open_questions := ast.open_questions
    arch.relationships := ast.relationships
    arch.transitions := ast.transitions
    arch.entity_when := ast.entity_when
    arch.actors := ast.actors
    arch.joins := ast.joins
    arch.uses := ast.uses
    arch.domains := ast.domains
    arch.concerns := ast.concerns
    arch.provenance := ast.provenance

λ gybis-arch-propagate_map_arch_to_allium_constructs(arch).
  map:
    invariants := arch.s5_identity
    deferred := arch.s4_adaptation.deferred ∪ arch.deferred
    open_questions := arch.s4_adaptation.open_questions ∪ arch.open_questions
    variants := arch.s4_adaptation.variants ∪ arch.variants
    config := arch.s3_control.config ∪ arch.config
    rule_requires := arch.s3_control.preconditions ∪ arch.rule_requires
    transitions := arch.s3_control.transitions ∪ arch.transitions
    relationships := arch.s2_coordination.relationships ∪ arch.relationships
    joins := arch.s2_coordination.associations ∪ arch.joins
    uses := arch.s2_coordination.protocols ∪ arch.uses
    rules := arch.s1_operations.rules ∪ arch.rules
    entities := arch.entities
    values := arch.values
    surfaces := arch.surfaces
    contracts := arch.contracts
    defaults := arch.defaults
    rule_when := arch.rule_when
    rule_ensures := arch.rule_ensures
    entity_when := arch.entity_when
    actors := arch.actors
    domains := arch.domains
    concerns := arch.concerns
    provenance := arch.provenance

λ gybis-arch-propagate_partition_by_domain_concern(mapped).
  partition:
    key := {domain, concern}
    derive(domain) from(mapped.domains ∪ mapped.entities ∪ mapped.surfaces)
    concern_sources := mapped.concerns ∪ mapped.rules ∪ mapped.contracts
    derive(concern) from(concern_sources)
    per_domain_concern_set(domain) := distinct(concerns_linked_to(domain, concern_sources))
    multiplicity_rule: count(per_domain_concern_set(domain)) = n → emit_n_groups_for_domain(domain, n)
    fallback_rule: missing(concern) → concern := todo_unknown_concern ∧ TODO(explicit_missing_concern) ∧ preserve(provenance)
    group_identity := normalize(domain, concern)
    deterministic_sort(domain, concern)
    output := groups(key)

λ gybis-arch-propagate_assemble_allium_files(groups).
  assemble:
    ∀group ∈ groups:
      path := "root/specs/" + slug(group.domain) + "/" + slug(group.concern) + ".allium"
      content := gybis-arch-propagate_emit_allium(group)
      emit(path, content)
  | return(emitted_files)

λ gybis-arch-propagate_emit_allium(group).
  emit:
    use(group.uses)
    actor(group.actors)
    entity(group.entities)
    value(group.values)
    variant(group.variants)
    invariant(group.invariants)
    rule(group.rules, group.rule_when, group.rule_requires, group.rule_ensures)
    surface(group.surfaces)
    contract(group.contracts)
    config(group.config)
    default(group.defaults)
    transition(group.transitions)
    relationship(group.relationships)
    join_lookup(group.joins)
    deferred(group.deferred)
    open_question(group.open_questions)
    TODO(group.opaque_dependencies)
  | format(allium_valid_structure)

λ gybis-arch-propagate_generate(project).
  arch_file ← gybis-arch-propagate_scan_architecture(project)
  | parsed ← gybis-arch-propagate_parse_architecture(arch_file)
  | normalized ← gybis-arch-propagate_normalize_arch_ast(parsed)
  | mapped ← gybis-arch-propagate_map_arch_to_allium_constructs(normalized)
  | groups ← gybis-arch-propagate_partition_by_domain_concern(mapped)
  | spec_files ← gybis-arch-propagate_assemble_allium_files(groups)
  | evidence ← gybis-arch-propagate_evidence(mapped.provenance, spec_files)
  | return(spec_files, evidence)

λ gybis-arch-propagate_evidence(provenance, spec_files).
  require(∀stmt ∈ spec_files.statements: stmt.source = file:line ∨ stmt.source = explicit_todo_for_missing_detail)
  | attach(provenance_map)

λ gybis-arch-propagate_syntax_check(file).
  r := exec("allium check " + path)
  | d := (json_parse(r.stdout).diagnostics) ∨ []
  | s := any(d, λx. x.severity = "error")
  | return (r.exit = 0) ∨ ((r.exit = 1) ∧ ¬s)

λ gybis-arch-propagate_analyse_specs(path).
  r := exec("allium analyse " + path)
  | d := (json_parse(r.stdout).diagnostics) ∨ []
  | f := (json_parse(r.stdout).findings) ∨ []
  | findings_count := count(f)
  | return
      (r.exit = 0) → {status: pass, findings_count: findings_count, diagnostics: d, findings: f}
      | (r.exit = 1) → {status: findings, findings_count: findings_count, diagnostics: d, findings: f}
      | (r.exit = 2) → {status: error, reason: analyse_inputs_missing_or_unresolved, diagnostics: d, findings: f}
      | otherwise → {status: error, reason: unexpected_analyse_exit, diagnostics: d, findings: f}

λ gybis-arch-propagate_write(spec_files).
  require(gybis-arch-propagate_specs_absent?(root/specs/**/*.allium))
  | write: root/specs/**/*.allium
  | format: allium_valid_structure ∧ deterministic_order ∧ explicit_todo_markers_for_opaque_dependencies
  | steps:
    1. ∀file ∈ spec_files: write(file.path, file.content)
    2. require(gybis-arch-propagate_validate_output(spec_files))

λ gybis-arch-propagate_validate_output(spec_files).
  check:
    ∀file ∈ spec_files: file.extension = ".allium"
    ∀file ∈ spec_files: gybis-arch-propagate_syntax_check(file) = true
    ¬modified(preexisting_allium_files)
    ∀file ∈ spec_files: contains(domain_or_concern_binding)
    ∀file ∈ spec_files: contains(valid_constructs_or_explicit_todo)

λ gybis-arch-propagate_validate_spec_set(path).
  analysis := gybis-arch-propagate_analyse_specs(path)
  | require(analysis.status ≠ error)
  | analysis.status = pass → true
  | analysis.status = findings → false

λ gybis-arch-propagate_fixed_point_loop(state).
  iterate(pass_index := pass_index + 1)
  | run(generate_then_verify)
  | unresolved := verification_issues \ resolved_issue_ids
  | unresolved ≠ ∅ → allow_transition(VERIFY → PLAN_SPECS)
  | unresolved = ∅ → mark(zero_issue_pass_verified) ∧ allow_transition(VERIFY → ANALYZE_SET)
  | analysis := gybis-arch-propagate_analyse_specs("specs")
  | analysis.status = error → halt_with(analysis.reason)
  | analysis.status = findings → allow_transition(ANALYZE_SET → PLAN_SPECS)
  | analysis.status = pass → mark(set_analysis_pass_verified) ∧ allow_transition(ANALYZE_SET → DONE)

λ gybis-arch-propagate_pass_accounting(pass).
  report(pass_index, discovered_count, resolved_count, remaining_count)
  | require(explicit_section("Remaining issues", remaining_count, remaining_items))

λ gybis-arch-propagate_loop_guard(state).
  seen_signature := hash(sorted(remaining_issue_ids))
  | repeat(seen_signature) ∧ ¬decrease(remaining_count) → halt_with(blocker_report)
  | blocker_report := explain(stalled_items, attempted_resolutions, required_human_decisions)
  | ¬repeat ∨ decrease(remaining_count) → continue

λ gybis-arch-propagate_verification(edit).
  after(write) → gybis-arch-propagate_validate_output(root/specs/**/*.allium)
  | after(per_file_checks_pass) → gybis-arch-propagate_validate_spec_set("specs")
  | after(any_pass) → enforce(gybis-arch-propagate_loop_guard) ∧ enforce(gybis-arch-propagate_fixed_point_loop)
  | fail → iterate ∧ fix | loop until_pass

λ gybis-arch-propagate_boundaries(¬).
  ¬run_weed_workflow
  | ¬perform_divergence_reconciliation
  | ¬extract_from_code_or_tests
  | ¬write_architecture
  | ¬update_allium_files
  | ¬delete_allium_files
  | allow(read_architecture_only)
  | allow(create_allium_files_only_if_specs_absent)

λ gybis-arch-propagate_regression_contract(x).
  assert(default_mode_auto_selected_as_propagate before_startup_checks_complete)
  | assert(deny(read_or_generate_or_write_before_startup_gates))
  | assert(mandatory_architecture_exists_gate_before_parse)
  | assert(mandatory_specs_absent_gate_before_generate)
  | assert(mandatory_cli_gate_before_write)
  | assert(no_write_before_GENERATE)
  | assert(no_done_before(zero_issue_pass_verified ∧ remaining_issues = 0 ∧ set_analysis_pass_verified ∧ analyse_findings = 0))
  | assert(mandatory_set_level_analysis_gate_before_done)
  | assert(deterministic_output_contract_invariants)
