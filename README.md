# gybis /JY-bis/

<img src="gybis-logo.png" alt="gybis logo" width="110" style="margin-bottom: 1em;" />

**A Developer-Command-Driven AI-Assisted Spec-Driven Software Development (SDD) Stack.**

## Overview

**gybis** integrates developer commands for AI-assisted specification-driven development into repositories that support bundled `.agents/skills/` directories.

The project combines:

- [Nucleus](https://github.com/michaelwhitford/nucleus) mathematical notation and VSM concepts
- [Allium](https://github.com/juxt/allium) behavioral specifications
- [Mementum](https://github.com/michaelwhitford/mementum) project-owned session memory
- Requirements, vocabulary, architecture, specification, test, code, and documentation workflows

## Gybis Documentation

For workflows, command families, use cases, stage accounting, architecture principles, memory operations, installation guidance, and upgrade procedures, see [gybis/GYBIS-README.md](gybis/GYBIS-README.md).

This is the single detailed guide for using and understanding gybis.

## Installation

From the target repository, copy the complete gybis bundle:

```bash
cp -ra <pathToGybisDirectory>/gybis/. . # e.g., cp -ra ~/Downloads/gybis/gybis/. .
```

This copies the hidden `.agents/skills/` command implementation and the repository bundles.

Do not use the full installation copy against an existing gybis installation. Existing project-owned files such as `mementum/state.md`, `mementum/index.md`, and local documentation must not be silently overwritten.

## Upgrading an Existing Installation

Update only the distributed command bundle:

```bash
rm -rf .agents/skills/gybis-* .agents/skills/internal && cp -ra <pathToGybisDirectory>/gybis/.agents/skills/. .agents/skills/
```

This removal and copy update gybis-owned skills only. It preserves skills owned by other tools.

Do not replace project specifications, source code, tests, or `mementum/` as part of the command-bundle update. After updating the bundle, initialize with `/gybis-init` and run `/gybis-memory-migrate` only if migration is required.

## Versioning

gybis follows [Clojure's](https://github.com/clojure/clojure) versioning philosophy by prioritizing stability, backward compatibility, and minimal breakage over rapid evolution or strict semantic-versioning adherence.

- **Strong emphasis on backward compatibility:** Development avoids breaking changes whenever possible.
- **No fixed roadmap:** Development remains open-ended.
- **Careful deprecation:** Deprecated behavior is handled transparently and with migration consideration.

## For gybis Developers: Upstream Derivation

This repository is an integration layer over several upstream projects. The content under `gybis/` is derived from pinned upstream commits, then adapted into a coherent developer-command-driven stack.

The distributed gybis bundle ships under `.agents/skills/`. This repository uses the same layout during local development. gybis is not a direct mirror of any one upstream repository.

### General Derivation Approach

1. Pin each upstream repository to a specific commit.
2. Select canonical upstream artifacts, language references, protocol references, and models.
3. Transform those artifacts into gybis conventions such as skills and internal references.
4. Publish the derived result in `gybis/` for downstream integration.

### Per-Upstream Transformations

| Upstream | Pinned commit | Source consumed | Transformation into gybis |
| --- | --- | --- | --- |
| [Allium](https://github.com/juxt/allium) | `527cd52` | Language semantics and behavioral-spec structure | Curated into Allium language references and encoded into specification skills |
| [Allium Tools](https://github.com/juxt/allium-tools) | `08d3139` | CLI validation and analysis capabilities | Executed by specification skills; user dependency only |
| [grill-with-docs](https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs) | `0ab1b63` | Grilling skill and dependencies | Used to derive the gybis requirements elicitation skill |
| [Mementum](https://github.com/michaelwhitford/mementum) | `4968400` | Mementum protocol semantics | Used to derive gybis memory skills |
| [Nucleus](https://github.com/michaelwhitford/nucleus) | `64880ed` | Nucleus notation, VSM model, and Lambda Compiler semantics | Used to derive gybis skills |

### Maintainer Notes

#### Nucleus Lambda Compiler

gybis relies on lambda notation and operator semantics defined by the upstream Nucleus Lambda Compiler. The compiler is a semantic source of truth even though it is not copied into gybis.

When a Nucleus pin changes, compare the updated Lambda Compiler semantics against derived gybis skills and revalidate lambda forms, operators, and workflow conventions.

When any upstream pin changes:

1. Update the pin.
2. Update the derived files.
3. Verify the corresponding skill behavior.
4. Revalidate the distributed bundle before publishing.

## License

AGPL 3.0

Copyright 2026 Paul Whittington

## Upstream Citations

### Allium

This project incorporates ideas, code, and/or structure from [Allium](https://github.com/juxt/allium) by JUXT.

Original project: https://github.com/juxt/allium

Portions derived from Allium remain subject to the terms of its license.

### Allium Tools

This project incorporates ideas, code, and/or structure from [Allium Tools](https://github.com/juxt/allium-tools) by JUXT.

Original project: https://github.com/juxt/allium-tools

Portions derived from Allium Tools remain subject to the terms of their license.

### Grill with Docs

This project incorporates ideas, code, and/or structure from [grill-with-docs](https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs) by Matt Pocock.

Original project: https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs

Portions derived from grill-with-docs remain subject to the terms of its license.

### Mementum

This project incorporates ideas, code, and/or structure from [Mementum](https://github.com/michaelwhitford/mementum) by Michael Whitford.

Original project: https://github.com/michaelwhitford/mementum

Portions derived from Mementum remain subject to the terms of its license.

### Nucleus

This project incorporates ideas, code, and/or structure from [Nucleus](https://github.com/michaelwhitford/nucleus) by Michael Whitford.

Original project: https://github.com/michaelwhitford/nucleus

Portions derived from Nucleus remain subject to the terms of its license.
