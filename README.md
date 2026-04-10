# gybis /JY-bis/ 

<img src="gybis-logo.png" alt="gybis logo" width="110" style="margin-bottom: 1em;" />



**An AI-assisted Spec-Driven Software Development (SDD) Stack.**

## Obligatory Word Definitions

**Gyre** /jahy<sup>uh</sup>r/

* a large-scale, coherent, rotating or spiraling flow

**Orbis** /ˈɔr.bis/

* a closed or periodic trajectory in a dynamical system, representing the set of all points reachable from an initial point under the system's update rules

## Overview

**gybis** is an AI-driven meta-repo (a repository of repositories) that generates a collection of AI artifacts, which when integrated  into a new/existing repository, adds an **AI-assisted Spec-Driven Software Development (SDD)** stack to the new/existing repository. 

The goal of **gybis** is to make it easy for developers to set up, utilize, and maintain an AI-assisted, command-driven SDD workflow in their repository by providing a comprehensive set of resources and suggested usage practices. 


## What is SDD and why gybis?

**Spec-Driven Development** is a practice where architectural and behavioral specifications of what a system is and what it does drive the implementation of how it does it. A specification of the system's architecture provides the context for the specification of its behaviors, and the specification of its behaviors provides the context for its implementation. This creates a virtuous cycle where architecture, behavior and implementation inform and constrain one another as the system evolves. In SDD, specifications are the source of truth for the system's intended behavior, and implementation is done by AI, with full developer visibility, to realize that behavior. Specs are written before or alongside implementation, kept in source control, and used directly to guide/generate implementation, guide/generate tests, and detect architectural/behavioral/implementation drift.

**gybis** adds AI-assistance to SDD by adding AI/Developer conversation to the entire workflow. All phases of the software development workflow are verified, harmonized and accelerated by AI assistance, while at the same time, making all phases of the workflow transparent and accessible to the developer. The AI is a collaborator that can be consulted at any time, but the developer is always in control of the process and the final decisions.

**gybis** provides the scaffolding to make AI-assisted SDD practical:

- **AI base context**: mathematical notation engages
  the AI model's structured reasoning rather than its conversational defaults. Its inherent precision means significantly fewer hallucinations. Its inherent density means significantly fewer tokens to convey the same context.
- **Architecture model**: an AI generated and maintained 5-layer architectural specification, stored in a file, that AI keeps up to date as projects evolve.
- **Behavioral Domain Specific Language (DSL)**: a behavioral specification language the AI can read and write precisely, not
  pseudocode, not free-form prose, but a structured DSL with rules, triggers, surfaces, and transition graphs that AI can reason about directly and minimize hallucinations. Behavioral specifications are saved in files, typically one per feature or module (e.g., orders, payments).
- **Persistent memory**: decisions, patterns, and insights are stored in files and recalled during AI sessions, so previous context is available between sessions.
