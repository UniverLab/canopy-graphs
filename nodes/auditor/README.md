# auditor

What this node is for: the review-and-repair step of a spec loop. It
reviews the change strictly against the spec, repairs what it objects
to in the working tree, runs the gate script once, and never commits —
a check node commits the repairs after the gates pass.

What it reads: `{{spec_content}}`, `{{previous_feedback}}`, and the
spec notes, where the `## Designer` note is the plan. Its own spec note
is a progress log it continues from on a retry.

What it leaves behind: its repairs in the working tree, uncommitted,
and a spec note of what it checked, fixed and left.

Config: `prompt_template`, the `Auditor` prompt of
`adversarial-developer`, bundled verbatim in `node.json`. No `platform`,
`model`, or `cli` bindings.

What the installer must fill in: a platform and a model at install
time, and the gate script: the prompt quotes `<GATE-SCRIPT>`, which
must become the command that runs the project's own gates.

Install:

```sh
canopy node install auditor
```
