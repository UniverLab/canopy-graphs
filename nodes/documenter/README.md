# documenter

What this node is for: the documentation step of a spec loop, after the
code is finished and green. It writes the user docs the change needs
and leaves them uncommitted for the commit node. It never writes or
changes production code.

What it reads: `{{spec_content}}` and the change itself, found with
`git status --short` and `git diff HEAD`. Untracked files are part of
the change and `git diff` hides them, so it reads new files directly.

What it leaves behind: the updated pages under `docs/` and the
project's README, uncommitted in the working tree.

Config: `prompt_template`, the `Documenter` prompt of
`canopy-loop-plain`, bundled verbatim in `node.json`. No `platform`,
`model`, or `cli` bindings.

What the installer must fill in: a platform and a model at install
time; nothing else.

Install:

```sh
canopy node install documenter
```
