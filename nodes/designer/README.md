# designer

What this node is for: the designer step of a spec loop. It
investigates the spec against the code and saves an executable plan as
its spec note. It never writes code and never commits.

What it reads: `{{spec_content}}` and the spec notes of the run. On a
retry it continues its own `## Designer` note instead of starting over,
and the note is shared with every later node.

What it leaves behind: the plan, saved with `spec_note_write` as its
spec note. That note is the deliverable; no file in the working tree
changes.

Config: `prompt_template`, the `Designer` prompt of
`adversarial-developer`, bundled verbatim in `node.json`. No `platform`,
`model`, or `cli` bindings.

What the installer must fill in: a platform and a model at install
time; nothing else.

Install:

```sh
canopy node install designer
```
