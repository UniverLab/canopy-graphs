---
title: Reviewer fixes
summary: The reviewer who rejects the work fixes it, instead of writing change requests for another model to read.
status: in use
implements:
  - node:reviewer-fixer
figure: reviewer-fixes.figure.json
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

Repair wins when the defect is small and clear: a missed edge case, a wrong boundary, a test that asserts too little. The reviewer already holds the full context, so the fix is one edit, not a handoff.

Bounce the work back when the defect is large or the implementer misread the spec. A reviewer that rewrites half the diff is no longer reviewing; it is implementing without the implementer's instructions, and its output deserves its own review.

## What we saw

The lab has no recorded origin story for this pattern, and no measurement of how often reviewer repairs beat a rejection. Treat the claim above as a design intent, not a result.

The practical rule is trust calibration. Give the repair licence to a reviewer model strong enough to edit carefully, and keep the bounce loop behind it for everything else.

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

The single run at the end is deliberate. The reviewer proves its own
repairs without turning into a second implementer that iterates.

This differs from the catalogue's `reviewer-committer`, which fails a
bad diff back to the implementer with a list of what is wrong instead
of fixing it. `canopy-loop` splits the two jobs: its first reviewer
only reports what is missing, its reviewer of record repairs what it
finds, and a check node commits those repairs. Pick the arrangement
that matches how much you trust your reviewer model.

## Try it

```sh
canopy node install reviewer-fixer
```
