---
title: Reviewer fixes
summary: The reviewer who rejects the work fixes it, instead of writing change requests for another model to read.
status: in use
implements:
  - node:reviewer-fixer
sources:
  - title: Building effective agents (Anthropic), evaluator-optimizer workflow
    url: https://www.anthropic.com/engineering/building-effective-agents
---

# Reviewer fixes

## The problem

A rejection is a message from one model to another. The implementer must read it, understand it and redo the work, and on a cheap model each round trip can lose detail or introduce a new fault. When the defect is small, the round trip costs more than the fix.

## The pattern

Let the reviewer repair what it rejects. Instead of a list of change requests, it edits the diff, and a later step commits. The loop back to the implementer is kept for the cases the reviewer cannot repair.

This differs from the plain evaluator-optimizer loop Anthropic describes, in which "one LLM call generates a response while another provides evaluation and feedback in a loop." Here the evaluator can also write.

## What we saw

The lab has no recorded origin story for this pattern, and no measurement of how often reviewer repairs beat a rejection. Treat the claim above as a design intent, not a result.

## In Canopy

The catalogue's `reviewer-fixer` node is the reviewer of record with
a licence to edit. Its preset reviews the diff strictly against the
spec, repairs what it objects to directly in the working tree,
rewrites tests that would not fail without the feature, and runs the
gate script once at the end. It never commits: a later node commits
its fixes.

The gate script path is the one thing the preset does not carry. Fill
in `<GATE-SCRIPT>` at install time with the project's deterministic
gates. From the preset in `nodes/reviewer-fixer/node.json`:

```
If you changed anything, run the gate script once at the end:
<GATE-SCRIPT>. It is exactly what the gate node after you runs. Fix
until it passes. If you changed nothing, do not run it.
```

This differs from the catalogue's `reviewer-committer`, which fails a
bad diff back to the implementer with a list of what is wrong instead
of fixing it. The `single-committer` graph goes further the other way:
its committer is told not to edit files at all. Pick the one that
matches how much you trust your reviewer model.

## Try it

```sh
canopy node install reviewer-fixer
```
