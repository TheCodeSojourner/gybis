💡 keep root repo local skill subset instead of mirroring bundled distribution

## Context
The root repo intentionally keeps a narrower local `.agents/skills` set rather than mirroring the broader distributable bundle in `gybis/.agents/skills`.

## Preference
- The root repo should keep its older local subset of skills.
- The bundled `gybis/.agents/skills` directory is authoritative for downstream distribution, not a forced sync target for this repo root.
- Do not run `rsync -a --delete gybis/.agents/ .agents/` in this repo without explicit approval.
- Root-local skill changes should stay scoped to the local development subset unless the project intentionally decides to broaden it.

## Feed-Forward
This is a repo-local convention, not a general rule for bundled installs. Preserve the distinction between local repo tooling and distributed bundle content.
