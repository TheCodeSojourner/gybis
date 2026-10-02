---
type: Insight
symbol: 🔁
title: Normalize validator results at stage boundaries
related: requirements-layer-family, skill-contract-system
---

When one skill uses another validator's result to decide stage readiness, normalize the result at the consumer boundary instead of assuming a shared raw shape.

Use a compact contract such as `{artifact_present, valid, errors}`. Map structured reports, boolean gates, and test-run outcomes into that shape; capture validator halts as invalid-stage results so one malformed downstream artifact does not abort checks for other stages. Treat unknown or missing evidence as pending, not valid. Keep warnings distinct from errors according to the owning check's policy.

This prevents readiness logic from depending on report prose or one validator's private output format.