---
type: Decision
symbol: 🎯
title: Reference-corpus cleanup: rename, delete, repair
related: weed-validity-gating-complete
---

Reference-corpus cleanup in the same session as the weed-validity gating.

- `internal/reference/allium-recommended-loops.md` → `recommended-loops.md`. Its content is gybis workflow protocol (entry gates, loop phases, convergence invariant), not Allium language, so the `allium-` prefix misdescribed it; `vsm-guide.md` already set the unprefixed convention. Git history confirms the unprefixed name was original — the prefix was added later in f1e0e29.
- `internal/reference/allium-construct-coverage.md` deleted: a 74-line orphan with zero consumers, not in the ref-check manifest (unverified), so it could silently rot.
- `gybis-req-elicit` had a silently-broken reference path: `preload: [internal/reference/recommended-loops]` pointed at a nonexistent file, and ref-check passed because it only verifies the manifest. Corrected to `read(internal/reference/recommended-loops.md) → loops_ref`.

The reference directory is now 8 files, each named for its subject, listed in the manifest, and read by at least one skill.
