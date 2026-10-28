# full-gates-cargo

What this node is for: the Rust full gate. One check command runs the
formatter in check mode, the linter at `-D warnings` and the whole test
suite; its exit code is the verdict, never a model's opinion.

What it reads: the repository only. It is a check node, so it carries a
command instead of a prompt: `{{spec_content}}`, `{{previous_feedback}}`
and the spec notes do not apply to it. Its output, pass or fail, is
what the next node routes on, and a failure's text reaches the next
prompt as feedback.

What it leaves behind: nothing on disk — it changes no file. It only
reports whether the change holds.

Config: the `command`, plus `mutex: ["cpu"]` so two graphs do not
compile at once, and `timeout_seconds: 6000` — the same config the
`Full gates` node carries in `canopy-loop-plain`. No `platform`,
`model`, or `cli` bindings.

What the installer must fill in: nothing for a Rust project. The
project should add its dependency audit and coverage floor to the
command, or replace it with its own gate script.

Install:

```sh
canopy node install full-gates-cargo
```
