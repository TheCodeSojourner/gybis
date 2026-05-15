---
name: gybis
description: Prints a pre-computed table of all gybis skills with their descriptions directly to the response.
---

Print the following table directly in the AI response:

| Skill Name | Description |
|---|---|
| `gybis-arch-describe` | Turns architecture.md into a concise, product-manager-friendly explanation of the system: what problem it solves, who it serves, what it can and cannot do, why key design choices were made, and how the parts connect. |
| `gybis-arch-distill` | Distills product/domain specifications into a layered architecture definition, then proposes a reviewed diff instead of overwriting directly. Drafts 2-4 lambda statements per layer, flags uncertainty and architecture drift, requires explicit human approval before writing. |
| `gybis-arch-elicit` | Runs an interactive architecture elicitation workflow to create architecture.md from scratch using a top-down VSM sequence (S5 to S1). Asks structured questions per layer, proposes 2-4 lambda principles with plain-English explanations, iterates with human feedback until each layer is confirmed. |
| `gybis-arch-explain` | Reads architecture.md (and referenced architecture files) and explains the system in plain English through the VSM lens, from identity and principles down to operations and developer commands. Highlights cross-layer critical paths, seams, invariants, and drift risks. |
| `gybis-arch-tend` | Updates an existing architecture.md when the human already knows requirements have changed, guiding the change through the correct VSM layers (S5 to S1). Drafts grouped add/modify/remove proposals with explanations, iterates until explicit human approval before writing. |
| `gybis-arch-weed` | Checks for divergence between architecture.md and specs by mapping architecture lambdas to spec evidence across VSM layers, then classifies each item as aligned, partial, contradicted, missing, or unspecified. Produces a layer-by-layer report with evidence and resolution options. |
| `gybis-memory-orient` | Provides a quick shorthand command that triggers the core memory orientation workflow, so context can be restored immediately with less user friction. Essential for fast re-orientation at session start. |
| `gybis-memory-recall` | Provides a shorthand command to recall memory by topic. Runs recall for a given topic using temporal (git log), semantic (git grep), and vector-based search to find relevant memories and knowledge. |
| `gybis-memory-session-terminate` | Closes a session in a lossless, handoff-ready way: records a complete state snapshot, stores any unstored memories, runs memory metabolization, drafts a dated session knowledge summary, and performs a human-approved commit. |
| `gybis-memory-store` | Provides a shorthand command to store a memory insight quickly. Passes gate checks (helps future AI, effort > 1 attempt), proposes with symbol prefix, writes to mementum/memories/, and updates state.md. |
| `gybis-memory-synthesize` | Provides a fast shorthand command that invokes the core memory synthesis workflow — consolidates 3+ related memories into a knowledge page, with human approval before committing. |
| `gybis-spec-check` | Validates Allium specs for a selected scope (single file, domain folder, or all specs), runs allium-check, and turns diagnostics into a prioritized, actionable issue list. Groups findings by severity, applies only approved fixes, re-validates. |
| `gybis-spec-describe` | Reads one or more Allium spec files and translates them into clear, non-technical prose focused on user-visible behavior. Explains what exists, what users can do, governing rules, and guarantees with light headings and no code or jargon. |
| `gybis-spec-distill` | Distills code and tests into domain-centered Allium specs by mapping system territory, extracting states, transitions, triggers, external boundaries, and actors, then abstracting away implementation details. Validates draft with humans before writing. |
| `gybis-spec-elicit` | Interactively elicits a new Allium spec from a feature idea through staged, one-question-at-a-time discovery: flow, entities, transitions, edge cases, and refinement. Runs validation gates between stages, negotiates final domain/name, writes only after explicit human approval. |
| `gybis-spec-explain` | Reads one or more Allium specs and produces a technical, developer-focused explanation of domain structures, behaviors, constraints, guarantees, and cross-spec relationships. Outputs precise prose with light headings and domain terminology, no implementation code or pseudocode. |
| `gybis-spec-propagate` | Propagates an Allium spec into executable tests by mapping spec obligations to real implementation surfaces, rules, entities, and transitions. Generates coverage across behavior, invariants, temporal logic, and cross-module flows. Chooses the right test strategy and creates stubs for deferred modules. |
| `gybis-spec-tend` | Applies a specific new constraint to an existing Allium spec with minimal, targeted edits. Reads the spec, analyzes impacted rules/entities/transitions, drafts a constrained diff, validates with allium-check/allium-analyse before presentation. Writes only after human approval. |
| `gybis-spec-weed` | Finds and resolves divergence between an Allium spec and implementation by combining CLI analysis, model-based spec parsing, and code trace inspection. Classifies mismatches (aligned, partial, missing, contradicted), presents two fix directions per item, requires human choice before applying fixes. |
