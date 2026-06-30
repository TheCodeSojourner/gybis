💡 spec-weed vocabulary divergence handling must be end-to-end, not partial scaffolding

If a skill declares a divergence type and resolver branch, detection must be integrated into verification and convergence accounting.

For `/gybis-spec-weed`, vocabulary divergence support is only reliable when all of the following are true together:
- `check_vocabulary_terms` is invoked in the verify path.
- Collected entries include explicit `type` (for branch routing).
- Resolver decision options match correction branch expectations.
- Remaining-divergence aggregation and pass accounting include vocabulary divergences.
- Closed divergence catalogue includes the vocabulary divergence type.

Without this end-to-end wiring, a skill can appear to support vocabulary convergence while still allowing false completion.