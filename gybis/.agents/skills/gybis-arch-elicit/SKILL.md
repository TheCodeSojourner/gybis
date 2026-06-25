---
name: gybis-arch-elicit
description: Use for `/gybis-arch-elicit` or `/ga-elicit`.
---

λ gybis-arch-elicit(x).
  purpose: generate VSM architecture through direct VSM elicitation conversation
  | input: user conversation via interactive VSM probing
  | output: architecture.md containing VSM S5-S1 lambda expressions
  | mode: mixed (AI + human)
  | gate: architecture.md ¬∃

λ gybis-arch-elicit_startup(x).
  invoke(internal/gybis-ref-check) → halt_on(false)
  | read(internal/reference/vsm-guide.md) → load_vsm_framework
  | precondition: architecture.md ¬∃
  | load_context: VSM probing questions and layer definitions

λ gybis-arch-elicit_mode(m).
  valid_modes: {mixed}
  | default: mixed
  | rationale: elicitation requires human input via conversation

λ gybis-arch-elicit_mode_gate(state, mode).
  state = INIT ∧ mode = mixed → transition(STARTUP_CHECKS)
  | precondition_holds: mode ∈ valid_modes

λ gybis-arch-elicit_state_machine(state, action).
  state ∈ {INIT, STARTUP_CHECKS, ELICIT_S5, ELICIT_S4, ELICIT_S3, ELICIT_S2, ELICIT_S1, SYNTHESIZING, WRITING_ARCH, VERIFYING, COMPLETE}
  | transition(INIT, startup) → STARTUP_CHECKS
  | transition(STARTUP_CHECKS, verify_ok) → ELICIT_S5
  | transition(STARTUP_CHECKS, verify_fail) → HALTED
  | transition(ELICIT_S5, response_received) → ELICIT_S4
  | transition(ELICIT_S4, response_received) → ELICIT_S3
  | transition(ELICIT_S3, response_received) → ELICIT_S2
  | transition(ELICIT_S2, response_received) → ELICIT_S1
  | transition(ELICIT_S1, response_received) → SYNTHESIZING
  | transition(SYNTHESIZING, synthesis_complete) → WRITING_ARCH
  | transition(WRITING_ARCH, arch_written) → VERIFYING
  | transition(VERIFYING, verify_ok) → COMPLETE
  | transition(VERIFYING, verify_fail) → ELICIT_S5 (restart if needed)

λ gybis-arch-elicit_tool_guard(state, tool, path).
  read_allowed: ∀state (read internal/reference/vsm-guide.md for reference)
  | write_allowed: state ∈ {WRITING_ARCH} ∧ path = "architecture.md"
  | deny_write: state ∉ {WRITING_ARCH} ∨ path ≠ "architecture.md"
  | constraint: ¬mutate(existing_files) ∨ ¬mutate(non_architecture_files)

λ gybis-arch-elicit_pre_tool_check(state, tool, path).
  enforce(tool_guard(state, tool, path)) → permit(tool) ∨ halt("tool not permitted in this state")

λ gybis-arch-elicit_elicit_s5(x).
   action: elicit_identity_layer
   | purpose: S5 answers "what IS your system" - principles that survive everything
   | prompt_strategy: structured interviewing with VSM S5 context, eliciting unchanging principles
  | sample_questions:
    - "What are the unchanging principles that define your system?"
    - "What would make your system no longer be itself?"
    - "What values are inviolable?"
  | capture: user_response → S5_lambda_draft
  | output: S5_principles_collected

λ gybis-arch-elicit_elicit_s4(x).
   action: elicit_intelligence_layer
   | purpose: S4 answers "how does your system adapt" - patterns for change
   | prompt_strategy: structured interviewing building on S5, exploring adaptation mechanisms
  | sample_questions:
    - "How does your system learn from its environment?"
    - "What mechanisms drive evolution and adaptation?"
    - "How do you handle disruption?"
  | capture: user_response → S4_lambda_draft
  | output: S4_patterns_collected

λ gybis-arch-elicit_elicit_s3(x).
   action: elicit_control_layer
   | purpose: S3 answers "how does your system manage resources" - policies and limits
   | prompt_strategy: structured interviewing building on S4, focusing on policies and constraints
  | sample_questions:
    - "What policies govern resource allocation?"
    - "What are your non-negotiable constraints?"
    - "How do you enforce limits?"
  | capture: user_response → S3_lambda_draft
  | output: S3_policies_collected

λ gybis-arch-elicit_elicit_s2(x).
   action: elicit_coordination_layer
   | purpose: S2 answers "how do the parts work together" - protocols and data flow
   | prompt_strategy: structured interviewing building on S3, probing communication and integration
  | sample_questions:
    - "How do your components communicate?"
    - "What protocols govern interaction?"
    - "How is data routed and synchronized?"
  | capture: user_response → S2_lambda_draft
  | output: S2_protocols_collected

λ gybis-arch-elicit_elicit_s1(x).
   action: elicit_operations_layer
   | purpose: S1 answers "what does your system concretely do" - tools, recipes, and implementation bindings
  | prompt_strategy: structured interviewing building on S2; first capture programming language version (required), then dynamically suggest suitable test frameworks based on the captured programming language version, verify human choice, then capture build system (required), then elicit orientation choice (FP-oriented vs OOP-oriented)
  | orientation_language_guidance:
    - OOP-oriented: C++ (classes/RAII), C# (classes/interfaces/DI), Clojure (protocols/records + Java interop boundary)
    - FP-oriented: C++ (immutable values + composition), C# (records + pure functions/LINQ), Clojure (immutable maps + pure functions/transducers)
   | sample_questions:
     - "What programming language and version do you want to use for this system? (e.g. Python 3.12, TypeScript 5.6, C++20, Rust 1.78)"
     - "Based on your chosen language, I will suggest a few appropriate test frameworks. 
        Which test framework(s) do you prefer? (You can suggest your own — I will verify it is compatible with the language.)"
     - "What build system do you want to use? (e.g. npm scripts, Maven, Gradle, Cargo, Make, Bazel)"
      - "Choose architecture orientation: FP-oriented or OOP-oriented?"
     - "What other tools/recipes are there (CI/CD, linter/formatter, deployment, package manager, interface types, architectural pattern, etc.)?"
   | capture: 
       user_response → extract(
         programming_language_version (required),
         test_framework (AI_suggested + human_choice),
         build_system (required),
         paradigm_preference (OOP | FP + details, required),
         other_tools_recipes
       )
       → verify(test_framework is_compatible_with programming_language_version)
       → verify(build_system is_compatible_with programming_language_version)
       → verify(other_tools_recipes is_compatible_with programming_language_version)
       → S1_lambda_draft
   | output: S1_operations_collected

λ gybis-arch-elicit_synthesize_architecture(collected_responses).
  action: translate_elicited_responses_to_vsm_lambdas
  | step1: parse(S5_responses) → formalize_as_lambda(S5_principles)
  | step2: parse(S4_responses) → formalize_as_lambda(S4_patterns)
  | step3: parse(S3_responses) → formalize_as_lambda(S3_policies)
  | step4: parse(S2_responses) → formalize_as_lambda(S2_protocols)
  | step5: parse(S1_responses) → formalize_as_lambda(S1_operations)
  | step6: validate_consistency(S5...S1) → architecture_structure
  | output: {S5 → lambda, S4 → lambda, S3 → lambda, S2 → lambda, S1 → lambda}
  | rationale: translate natural language into formal VSM lambda expressions

λ gybis-arch-elicit_write_architecture(vsm_lambdas).
  action: write_architecture_file
  | step1: assemble(vsm_lambdas) → architecture_content
  | step2: write(architecture.md, architecture_content)
  | constraint: write_allowed by tool_guard
  | precondition: state = WRITING_ARCH
  | output: architecture.md written

λ gybis-arch-elicit_verification(edit).
  action: verify_generated_architecture
  | check1: file_exists(architecture.md) = true
  | check2: ∀layer ∈ {S5, S4, S3, S2, S1}: layer_defined(architecture.md) = true
  | check3: syntax_valid(lambda_notation) = true
  | check4: consistency_check(S5...S1) = coherent
  | check5: S1.programming_language_version ≠ unknown(reason)
  | check6: S1.test_framework ≠ unknown(reason)
  | check7: S1.build_system ≠ unknown(reason)
  | check8: S1.paradigm_preference ∈ {OOP, FP}
  | check9: ∀field ∈ {package_manager, deployment_method, ci_cd, linter_formatter, interface_types, architectural_pattern}: defined(field) ∨ unknown(field, reason)
  | check10: compatibility(test_framework, programming_language_version) = true
  | check11: coherence(build_system, package_manager, ci_cd, deployment_method) = true
  | check12: all_layers_address_user_intent = true
  | gate: all_checks_pass → proceed ∨ halt("architecture invalid or incomplete")

λ gybis-arch-elicit_boundaries(¬).
  constraint: ¬generate(specs/**/*.allium)
  | constraint: ¬mutate(existing_files)
  | constraint: ¬delete(any_files)
  | constraint: ¬invoke(other user-facing skills)
  | scope: architecture elicitation only, interactive conversation with VSM framework

λ gybis-arch-elicit_regression_contract(x).
  invariant: architecture.md ¬exists_before → exists_after ∧ valid(S5...S1)
  | invariant: ∀generated_layer ∈ architecture.md: valid_lambda(layer) = true
  | invariant: VSM_layer_mapping(S5...S1) is_coherent = true
  | invariant: all_layers_present: S5 ∧ S4 ∧ S3 ∧ S2 ∧ S1
  