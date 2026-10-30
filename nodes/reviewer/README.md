# reviewer

What this node is for: the completeness review of a spec loop. It
locates every requirement at `file:line` and fails with the list of what
is missing or wrong. It asks for changes: it never fixes the code and
never commits.

What it reads: `{{spec_content}}`, `{{previous_feedback}}`, and the
spec notes, where the `## Designer` note is the plan the change had to
follow.

What it leaves behind: a report — one line per requirement with its
`file:line`, or a numbered list of what is MISSING — and no change in
the working tree.

Config: `prompt_template`, the `Reviewer` prompt of
`adversarial-developer`, bundled verbatim in `node.json`. No `platform`,
`model`, or `cli` bindings.

What the installer must fill in: a platform and a model at install
time; nothing else. This node does not run the gate script: a
deterministic gate has already run it on the same change.

Install:

```sh
canopy node install reviewer
```
