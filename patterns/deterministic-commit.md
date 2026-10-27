---
title: Deterministic commit
summary: A check node commits with the spec's own Conventional Commit subject, and a model never writes the commit.
status: in use
implements:
  - node:commit-check
  - node:reviewer-committer
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

Read this section carefully, because the catalogue is only part of the way there.

What exists today:

- `commit-check` is the proof step. It passes only if HEAD moved since the spec started. From `nodes/commit-check/node.json`:

```json
"config": {
  "command": "test \"$(git rev-parse HEAD)\" != \"{{spec_start_head}}\""
}
```

- `reviewer-committer` is the baseline the pattern replaces. A model reviews the diff and then commits, and its preset asks for a concise message with no trailers. The message is still model-written.

What does not exist yet: a catalogue node whose check command performs the commit with the spec's subject. Until then, the pattern is the lab's convention, and the two documents above cover its proof half and the model-committed version it improves on.

The stronger proof form is switched on in the node's README, and requires an earlier node with commit rights:

```sh
test -n "{{spec_committed_head}}" && test "$(git rev-parse HEAD)" = "{{spec_committed_head}}"
```

## Try it

```sh
canopy node install commit-check
canopy node install reviewer-committer
```
