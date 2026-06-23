💡 gybis is tool-agnostic — skills integrate into any AI tool supporting a `skills/` directory

## Context
Three recent commits (2026-06-23) clarified gybis positioning:
- gybis skills are NOT Cline-specific
- This repo uses `.cline/` as a local workspace convention during development
- The framework is designed to work with any compatible AI tool

## Key Distinction
- **User-facing gybis skills** (`/gybis-*` commands): Tool-agnostic. Integrate into any AI assistant with `skills/` support.
- **Developer Commands in this repo** (README.md Developer Commands section): Explicitly Cline-specific. This repo's development workflow uses Cline.
- **Local workspace**: `.cline/` is a local scratch convention; not part of the distributed `gybis/` bundle.

## Documentation Changes
- Overview: Changed from "Cline artifacts" to "AI-assistant skills that can be integrated into repositories through tools that support a `skills/` directory"
- User Commands: "Use them with any compatible AI tool configured to consume the gybis skills directory"
- Upstream Derivation: Added note that repo uses `.cline/` locally but gybis itself is tool-agnostic
- New section: Lambda Compiler Workspace explains `.cline/` is a local development convention

## Feed-Forward
When describing gybis to users/integrators: emphasize tool-agnostic nature. When describing this repo's dev setup: Cline-specific.