💡 Keep README and GYBIS-README complementary, not mirrored

When root README and gybis/GYBIS-README diverge, align behavior-critical semantics (workflow order, command availability, terminology, citations) but avoid wholesale section duplication.

Use this split:
- README.md: distribution context, broader project framing, maintainer derivation details, and short check/tend/weed philosophy/matrix sections.
- gybis/GYBIS-README.md: hands-on stack workflow, operational guidance, and more practical check/tend/weed cheat sheets.

Required sync points:
- Vocabulary-first flow must be consistent in both docs.
- Command names/aliases must match gybis-help canonical wording.
- Upstream citations should not be circular (no self-upstream entries).
- Check/tend/weed explanations should stay aligned, but the root README should remain shorter and more conceptual.

This preserves coherence for users while preventing documentation bloat and copy drift.