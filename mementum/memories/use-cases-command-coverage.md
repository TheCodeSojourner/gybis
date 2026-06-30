💡 Use Cases section should map scenarios to full command surface

When adding a Use Cases section to `gybis/GYBIS-README.md`, keep it scenario-driven and ensure every user-facing command is covered through either a direct use case or an explicit command pairing.

High-value clarifications to include in-place:
- `/gybis-init` is full session startup; `/gybis-memory-orient` is mid-session rehydration.
- `/gybis-memory-store` supports explicit input or prompt-for-insight mode.
- `/gybis-memory-synthesize` is cross-memory consolidation, not topic-specific recall.
- `tend` refines existing artifacts; `weed` resolves drift between layers; `distill` extracts initial artifacts from existing material.

This prevents command-table drift while keeping operational guidance concise and readable.