💡 hidden bundle installs need `cp -ra <bundle>/. .`

## Context
When gybis ships command implementations under the hidden `.agents/skills/` directory, shell globs like `gybis/*` skip that content during installation.

## Feed-Forward
If the distributed bundle includes hidden top-level paths, installation docs and automation must copy `gybis/.` instead of using `*`. Recheck shell-glob assumptions whenever bundle layout changes introduce dot-directories.