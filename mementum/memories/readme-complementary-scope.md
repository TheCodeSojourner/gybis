💡 Keep README and GYBIS-README complementary, not mirrored

When root README and gybis/GYBIS-README diverge, align behavior-critical semantics (workflow order, command availability, terminology, citations) but avoid wholesale section duplication.

Use this split:
- README.md: distribution context, broader project framing, maintainer derivation details.
- gybis/GYBIS-README.md: hands-on stack workflow and operational guidance.

Required sync points:
- Vocabulary-first flow must be consistent in both docs.
- Command names/aliases must match gybis-help canonical wording.
- Upstream citations should not be circular (no self-upstream entries).

This preserves coherence for users while preventing documentation bloat and copy drift.