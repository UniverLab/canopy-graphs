---
title: Reviewer fixes
summary: The reviewer who rejects the work fixes it, instead of writing change requests for another model to read.
status: in use
implements:
  - node:reviewer-committer
sources:
  - title: Building effective agents (Anthropic), evaluator-optimizer workflow
    url: https://www.anthropic.com/engineering/building-effective-agents
---

# Reviewer fixes

## The problem

A rejection is a message from one model to another. The implementer must read it, understand it and redo the work, and on a cheap model each round trip can lose detail or introduce a new fault. When the defect is small, the round trip costs more than the fix.

## The pattern

Let the reviewer repair what it rejects. Instead of a list of change requests, it edits the diff and then commits. The loop back to the implementer is kept for the cases the reviewer cannot repair.

This differs from the plain evaluator-optimizer loop Anthropic describes, in which "one LLM call generates a response while another provides evaluation and feedback in a loop." Here the evaluator can also write.

## What we saw

The lab has no recorded origin story for this pattern, and no measurement of how often reviewer repairs beat a rejection. Treat the claim above as a design intent, not a result.

## In Canopy

The catalogue's `reviewer-committer` node is the reviewer role with commit rights. Read it before you rely on it for this pattern: as shipped, its preset reviews the diff, makes exactly one commit when the spec is satisfied, and otherwise fails with a specific list of what is wrong so the implementer can address it. It does not edit files.

To get the fixing behaviour, install the node and change its preset so that the reviewer may edit the diff before committing. The node is the right place to start because it already carries commit rights and the strict review against the spec.

The graph gives it commit rights like this, from `graphs/spec-loop/graph.json`:

```json
{
  "name": "reviewer",
  "kind": "agent",
  "position": 5,
  "config": {
    "prompt_preset": "reviewer",
    "commit_rights": true
  }
}
```

The `single-committer` graph goes the other way: its committer is told not to edit files and to fail the diff back instead. Pick the one that matches how much you trust your reviewer model.

## Try it

```sh
canopy node install reviewer-committer
```

Then edit the preset in the installed node to allow the reviewer to fix the diff.
