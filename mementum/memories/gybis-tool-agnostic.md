💡 gybis is tool-agnostic — skills integrate into any AI tool supporting a `skills/` directory

## Context
Three recent commits (2026-06-23) clarified gybis positioning:
- gybis skills are NOT Cline-specific
- This repo uses `.agents/` as a local workspace convention during development
- The framework is designed to work with any compatible AI tool

## Key Distinction
- **User-facing gybis skills** (`/gybis-*` commands): Tool-agnostic. Integrate into any AI assistant with `skills/` support.
- **Developer Commands in this repo** (README.md Developer Commands section): Tooling-agnostic, backed by skills loaded from `.agents/`.
- **Distributed bundle**: gybis now ships its command implementations under `.agents/skills/`.

## Documentation Changes
- Overview: Changed from "Cline artifacts" to "AI-assistant skills that can be integrated into repositories through tools that support a `skills/` directory"
- User Commands: "Use them with any compatible AI tool configured to consume the gybis skills directory"
- Upstream Derivation: Updated note to reflect that the distributed bundle also ships under `.agents/skills/`
- New section: Lambda Compiler Workspace explains `.agents/` is a local development convention

## Feed-Forward
When describing gybis to users/integrators: emphasize tool-agnostic nature and note that the shipped bundle now places commands in `.agents/skills/`.