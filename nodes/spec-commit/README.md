# spec-commit

What this node is for: the deterministic commit step after a review.
It commits all changes with the spec's own commit message,
`$CANOPY_SPEC_COMMIT_MESSAGE`, and passes when there is nothing to
commit.

Config: the `command` plus `commit_rights: true`, so the engine
attributes the commit to this node. No `platform`, `model`, or `cli`
bindings.

What the installer must fill in: nothing for the basic form. Pushing
to the work branch is optional: if the graph should push after
committing, append the push to the command.

Install:

```sh
canopy node install spec-commit
```
