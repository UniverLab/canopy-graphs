# reviewer-fixer

What this node is for: the reviewer of record at the end of a spec
loop. It reviews the diff strictly against the spec, repairs what it
objects to directly in the working tree, and never commits — a later
node commits its fixes.

Config: `prompt_preset: reviewer-fixer`, with the preset bundled in
`node.json`. No `platform`, `model`, or `cli` bindings.

What the installer must fill in: the harness that runs this node
(platform/model) at install time, and the gate script path where the
preset reads `<GATE-SCRIPT>`; nothing else.

Install:

```sh
canopy node install reviewer-fixer
```
