# gybis /JY-bis/ 

<img src="gybis-logo.png" alt="gybis logo" width="110" style="margin-bottom: 1em;" />



**A Developer-Command-Driven AI-Assisted Spec-Driven Software Development (SDD) Stack.**

## Obligatory Word Definitions

**Gyre** /jahy<sup>uh</sup>r/

* a large-scale, coherent, rotating or spiraling flow

**Orbis** /ˈɔr.bis/

* a closed or periodic trajectory in a dynamical system, representing the set of all points reachable from an initial point under the system's update rules

## Overview

**gybis** contains a collection of [Cline](https://github.com/cline/cline) artifacts (e.g., rules, workflows), which when integrated into a new/existing repository, adds a **Developer-Command-Driven AI-Assisted Spec-Driven Software Development (SDD) Stack** to the new/existing repository. 

The goal of **gybis** is to make it easy for developers to set up, utilize, and maintain their projects using an SDD workflow by providing a comprehensive set of resources and suggested usage practices.

## What is SDD and why gybis?

**Spec-Driven Development** is a practice where architectural and behavioral specifications of `what` a system `is` and `what` it `does` drive the implementation of `how` it does it (e.g., code and tests). A specification of the system's `architecture` provides the context for the specification of its `behaviors`, and the specification of its `behaviors` provides the context for its `implementation` (e.g., code and tests). This creates a virtuous cycle where `architecture`, `behavior` and `implementation` inform and constrain one another as the system evolves. In SDD, specifications are the source of truth for the system's intended behavior, and implementation is done by AI, with full developer visibility. Specs are written before or alongside implementation, kept in source control, and used directly to guide/generate implementation, and detect architectural/behavioral/implementation drift.

**gybis** adds Developer-Command-Driven AI-Assistance to SDD by adding AI/Developer conversation to the entire workflow. All phases of the software development workflow are verified, harmonized and accelerated by AI assistance, while at the same time, making all phases of the workflow transparent and accessible to the developer. The AI is a collaborator that can be consulted at any time, but the developer is always in control of the process and the final decisions.

**gybis** provides the scaffolding to make AI-assisted SDD practical:

- **AI base context**: [Nucleus](https://github.com/michaelwhitford/nucleus) mathematical notation engages
  the AI model's structured reasoning rather than its conversational defaults. The inherent precision means significantly fewer hallucinations, and the inherent density means significantly fewer tokens to convey the same context.
- **Architecture model**: A derivative of the [Nucleus VSM](https://github.com/michaelwhitford/nucleus/blob/main/VSM.md) allows an AI to generate and maintain a 5-layer architectural specification, stored in a file, that AI keeps up to date as projects evolve.
- **Behavioral Domain Specific Language (DSL)**: The [Allium](https://github.com/juxt/allium) DSL is a behavioral specification language the AI can read and write precisely, not
  pseudocode, not free-form prose, but a structured DSL with rules, triggers, surfaces, and transition graphs that AI can reason about directly and minimize hallucinations. Behavioral specifications are saved in files, typically one per feature or module (e.g., orders, payments).
- **Persistent memory**: [Mementum](https://github.com/michaelwhitford/mementum) is used to manage decisions, patterns, and insights that are stored in files and recalled during AI sessions, so previous context is available between sessions.

## References

* [**allium**](https://github.com/juxt/allium) - Behavioral specification
* [**allium-tools**](https://github.com/juxt/allium-tools) - Behavioral specification tools
* [**babashka**](https://github.com/babashka/babashka) - Automation and scripting
* [**mementum**](https://github.com/michaelwhitford/mementum) - Persistent memory
* [**nucleus**](https://github.com/michaelwhitford/nucleus) - AI base context, and architectural specifications
