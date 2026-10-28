# resilient-implementer

What this node is for: implementing one spec with a fallback. Two
implementer members form a cascade: if the first fails on quota or
harness, the run falls through to the next instead of failing.

Config: the shared `prompt_template` (the implementer role verbatim),
`kind: cascade`, `min_pass: 1`, plus two unbound members. No
`platform`, `model`, or `cli` bindings anywhere.

What the installer must fill in: the harness per member
(platform/model) at install time; nothing else.

Install:

```sh
canopy node install resilient-implementer
```
