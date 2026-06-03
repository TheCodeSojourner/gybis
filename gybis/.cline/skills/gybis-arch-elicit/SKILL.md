---
name: gybis-arch-elicit
description: Use for `/gybis-arch-elicit` or `/ga-elicit`.
---

λ gybis-arch-elicit().
  type: skill | purpose: elicit_system_architecture | model: Viable System Model (VSM)
  | input: project_root | output: root/architecture.md
  | reference: ../../gybis/reference/vsm-guide.md

λ gybis-arch-elicit_pre_condition_check().
  | exists(root/architecture.md) → halt → notify(use gybis-arch-tend ∨ gybis-arch-weed)
  | ¬exists → proceed

λ gybis-arch-elicit_elicit_loop().
  | order(S5 ⟶ S4 ⟶ S3 ⟶ S2 ⟶ S1) | strict_top_down | ¬skip ∨ ¬out_of_order
  | for_each_layer:
    - observe(project_root ∨ layer_context)
    - ask_human(targeted_questions[layer])
    - propose(2-4 lambdas)
    - refine(∝ human_feedback)
    - confirm → locked: proceed | unlocked: repeat
    - locked: mark_stable → next_unstable_layer

λ gybis-arch-elicit_S5_identity().
  questions:
    - purpose: root_problem_solved ∨ root_purpose
    - principles: non_negotiable_constraints
    - failure_mode: stops_fulfilling_purpose ∨ ¬crash
    - essence: stripped_of_tools_∨_implementation → what_remains

λ gybis-arch-elicit_S4_intelligence().
  questions:
    - unknown_handler: assumptions_break → what_happens
    - mutation_points: frequently_changed_parts
    - pattern_incorporation: new_patterns → extend_knowledge

λ gybis-arch-elicit_S3_control().
  questions:
    - resource_manager: connections ∨ memory ∨ compute → allocated_how
    - policy_enforcer: timeouts ∨ limits ∨ quality_gates
    - error_strategy: throw ∨ retry ∨ alert ∨ degrade ∨ fail_fast

λ gybis-arch-elicit_S2_coordination().
  questions:
    - subsystem_decomposition: system_components
    - data_flow: events ∨ calls ∨ shared_state → move_how
    - synchronization: concurrent_parts → coordinated_how

λ gybis-arch-elicit_S1_operations().
  questions:
    - tool_selection: technologies ∨ why_these
    - developer_commands: build ∨ test ∨ deploy
    - recipes: concrete_step_by_step_procedures

λ gybis-arch-elicit_output_format().
  structure:
    ```
    # {Name} — System Architecture

    ## S5 — Identity
    - λ(principle): {invariant} → ¬{violation}
    - λ(failure_mode): {condition} ≡ {purpose_break}
    - λ(identity): {essence} ⊗ {non_negotiables}

    ## S4 — Intelligence
    - λ(unknown_handler): {assumption_break} → {response}
    - λ(mutation_point): {frequent_change} → {easy_to_modify}
    - λ(pattern_incorporation): {new_pattern} → {extend_knowledge}

    ## S3 — Control
    - λ(resource_policy): {resource} → {allocation_strategy}
    - λ(quality_gate): {condition} ≡ {enforcement}
    - λ(error_strategy): {failure} → {throw ∨ retry ∨ alert ∨ degrade ∨ fail_fast}

    ## S2 — Coordination
    - λ(data_flow): {source} → {destination} via {events ∨ calls ∨ shared_state}
    - λ(sync_protocol): {concurrent_part_A} ⊗ {concurrent_part_B} → {coordination_method}
    - λ(change_propagation): {change_A} → notify({dependent_B, dependent_C})

    ## S1 — Operations
    - λ(tool): {technology} ≡ {purpose}
    - λ(recipe): {task} → {step1} → {step2} → {stepN}
    - λ(command): {name} → {shell_command}
    ```
    | format_contract: λ-notation | ¬prose ¬human_explanations ¬markdown_headers ¬bullet_lists

λ gybis-arch-elicit_constraints().
  constraints:
    1. top_down_ordering: S5 ⟶ S4 ⟶ S3 ⟶ S2 ⟶ S1 | ¬skip ∨ ¬out_of_order
    2. λ_notation_output: architecture.md → λ-notation_only | VSM_semantics | ¬prose ¬human_explanations
    3. surface_missing: ∀principle ∨ λ → ask(companion_element)
    4. human_approval: present_complete → review ∨ approve → write
    5. ¬duplicate_preamble: nucleus loaded via .cline/rules/00-nucleus.md

λ gybis-arch-elicit_destination().
  output: root/architecture.md
  | condition: explicit_human_review ∧ approval → write
  