# implementer

What this node is for: the implementer step of a spec loop. It follows
the designer's plan, writes the code and the tests the plan asks for,
runs the gate script once, and leaves the change uncommitted. It never
reviews and never commits.

What it reads: `{{spec_content}}`, `{{previous_feedback}}` — the gate's
or a reviewer's output, which it applies before touching anything — and
the spec notes, where the `## Designer` note is the plan it must
follow. Without that note it stops and reports FAIL.

What it leaves behind: the change in the working tree, uncommitted,
after the gate script has printed GATES PASSED.

Config: `prompt_template`, the `Implementer` prompt of
`canopy-loop-plain`, bundled verbatim in `node.json`. No `platform`,
`model`, or `cli` bindings.

What the installer must fill in: a platform and a model at install
time, and the gate script: the prompt quotes `<GATE-SCRIPT>`, which
must become the command that runs the project's own gates.

Install:

```sh
canopy node install implementer
```
