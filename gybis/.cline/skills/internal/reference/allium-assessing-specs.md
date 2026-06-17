# Assessing specs

When working with an Allium spec, assess its maturity before deciding what to do next. Spec maturity isn't uniform — a well-developed entity with full rules and surfaces can sit alongside a newly sketched entity with just a transition graph, in the same file.

## Spec-level assessment

λ(construct, entities_no_transitions → lifecycles_unexplored)
λ(construct, transition_graphs → lifecycles_sketched(states_and_flows_known)
λ(construct, rules_witnessing_transitions → behaviour_specified(triggers_guards_outcomes)
λ(construct, surfaces_exposes_provides → boundaries_defined(who_sees_what)
λ(construct, actors_identified_by → roles_formalised)
λ(construct, invariants → cross_cutting_properties_asserted)
λ(construct, open_questions → known_unknowns_documented)
λ(construct, deferred_specifications → complexity_acked_scoped_for_later)

λ(assessment, coarse, entities + transitions ∧ ¬rules → fill(rules: "what triggers this transition?"))
λ(assessment, behaviourless, rules ∧ ¬surfaces → add(surfaces: "actors and what they see"))

## Per-entity assessment

λ(entity_checklist, transition_graph? ∧ all_witnessing_rules? ∧ all_surfaces_providing_triggers? ∧ all_requires_traceable_to_producer?)
λ(entity_complete, all_four → structurally_complete(¬exception_transitions ∧ ¬temporal_triggers ∧ ¬failure_paths → obstacle_elicitation))
λ(entity_missing_fourth, ¬traceable_requires → gaps_user_may_not_know)

## When to use `check` vs `analyse`

λ(when, check, available → run(after_each_edit, validates(syntax ∧ field_resolution ∧ transition_structure ∧ witnessing_rules) ∧ fast ∧ useful_at_all_stages))
λ(when, analyse, available → run(at_checkpoints: completeness_request ∧ entity_has(rules ∧ surfaces) ∧ entity_transition ∧ step_back_review, reasons_about(missing_producers ∧ dead_transitions ∧ deadlocks)))
λ(when, cli_missing, ¬CLI_available → fallback(language_reference_validation) ∧ note("CLI available via Homebrew/crates.io — see README"))
λ(when, analyse_unrecognized, analyse → unrecognised_command_error → CLI_predates_analyse_feature, fallback(conversational_analysis: trace_data_flow_reachability_by_reading_spec) ∧ ¬retry(analyse, same_session) ∧ note(update_CLI_enables_automated_checking))

## Adjusting your approach

λ(approach, coarse_entity → walkthrough("What triggers this transition? Who's involved?"))
λ(approach, detailed_entity → gap_analysis("This rule requires a value that nothing produces. Where from?"))
λ(approach, well_specified_entity → validation("Here's the lifecycle — does it match your mental model?"))
λ(approach, warning, ¬detailed_analysis_on_coarse_spec → noise_about_unwritten_things ∧ ¬exploratory_questions_on_complete_entity → user_already_answered_them)

## Communicating with stakeholders

Users are not expected to read or write Allium syntax. When discussing the spec with stakeholders, translate constructs into domain language:

- Instead of showing a transition graph, describe the lifecycle: "A candidacy starts as applied, moves through screening and interviewing, and ends as either hired or rejected."
- Instead of showing a rule, describe the behaviour: "When the recruiter advances a candidate, the system checks that the background check is clear before moving to interviews."
- Instead of showing a surface, describe the interaction: "The recruiter sees a queue of candidates awaiting screening, with their name and the role they applied for."
- Instead of listing `open_questions`, pose them directly: "One thing we haven't resolved — what happens to in-progress candidacies when a role is closed?"

When validating the spec, describe what it says and ask whether that matches expectations. Don't present the spec itself for review unless the user has shown they're comfortable reading it. The spec is the artefact; the conversation is in domain terms.