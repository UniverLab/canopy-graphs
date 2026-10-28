---
title: Deterministic commit
summary: A check node commits with the spec's own Conventional Commit subject, and a model never writes the commit.
status: in use
implements:
  - node:spec-commit
  - node:commit-check
sources:
  - title: Building effective agents (Anthropic), ground truth from the environment
    url: https://www.anthropic.com/engineering/building-effective-agents
---

# Deterministic commit

## The problem

A commit message written by a model drifts. It invents trailers, rewords the intent, or describes what the model thinks it did. The history then no longer matches the work that was asked for.

## The pattern

Let the spec name its own commit. The spec's name is a Conventional Commit subject, `<type>(<scope>): <subject>`, and a deterministic check node runs the commit with that text verbatim. A model never writes it.

A second check proves the commit landed. Both are facts about the repo, so neither needs a judge.

## What we saw

The lab names every spec as the subject its commit will carry, and keeps its graphs' commits verbatim from the spec name. The lab has no recorded origin story for it beyond that convention.

## In Canopy

Two catalogue documents carry this pattern: one commits, one proves.

`spec-commit` is the commit step. It is a check node with commit
rights whose command stages everything and commits with the spec's own
message verbatim. From `nodes/spec-commit/node.json`:

```json
"config": {
  "command": "if [ -z \"$(git status --porcelain)\" ]; then echo 'tree clean: nothing to commit'; exit 0; fi; if [ -z \"$CANOPY_SPEC_COMMIT_MESSAGE\" ]; then echo 'CANOPY_SPEC_COMMIT_MESSAGE is empty'; exit 1; fi; git add -A && git commit -q -m \"$CANOPY_SPEC_COMMIT_MESSAGE\" && echo \"committed $(git log -1 --oneline)\"",
  "commit_rights": true
}
```

When the tree is already clean, the command passes without committing.
Pushing to the work branch is optional; the node's README says where
to add the push.

`commit-check` is the proof step. It passes only if HEAD moved since
the spec started. From `nodes/commit-check/node.json`:

```json
"config": {
  "command": "test \"$(git rev-parse HEAD)\" != \"{{spec_start_head}}\""
}
```

The stronger proof form is switched on in the node's README, and requires an earlier node with commit rights:

```sh
test -n "{{spec_committed_head}}" && test "$(git rev-parse HEAD)" = "{{spec_committed_head}}"
```

The model-committed version this pattern replaces is
`reviewer-committer`: a model reviews the diff and then commits, with
a concise message it writes itself.

## Try it

```sh
canopy node install spec-commit
canopy node install commit-check
```
