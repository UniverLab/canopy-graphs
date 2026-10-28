---
title: Cross-platform check
summary: The reviewer runs on a different vendor from the implementer, so one family's blind spots and one quota ceiling cannot stall the graph.
status: in use
implements:
  - graph:spec-loop
figure: cross-platform-check.figure.json
sources:
  - title: Building effective agents (Anthropic), evaluator-optimizer workflow
    url: https://www.anthropic.com/engineering/building-effective-agents
---

# Cross-platform check

## The problem

A model reviewing its own family's output shares its blind spots. It learned the same habits, trips on the same cases, and reads the spec through the same lens as the author. A second opinion from the same family is barely a second opinion.

It also shares the author's quota ceiling. When the vendor subscription runs out, the writer and the judge go dark together, and the graph stalls with work waiting.

## The pattern

Keep the gated shape, but put the reviewer on a different vendor from the implementer. One family writes, another family judges.

Splitting the two buys independent judgment and means one exhausted subscription does not stall the whole graph. The implementer keeps drafting on its vendor while the reviewer keeps checking on another, and a failure of judgment in one family is caught by eyes trained elsewhere.

Keep the cross-check prompt narrow: the diff, the spec, and an instruction to find what is wrong. A cross-checker invited to re-implement will re-implement, and then there are two authors and no reviewer.

Use it for security-sensitive changes, and anywhere the fix looks overfit to one model's reading of the spec.

## What we saw

The lab runs its reviews across vendors as standard practice. When the implementer and the reviewer come from different families, each covers readings the other misses, and a single vendor outage or quota ceiling leaves half the loop still standing.

The lab has not measured how many defects this catches that a same-family review would miss. It is a design choice, not a benchmark.

## In Canopy

No catalogue document is dedicated to this pattern, because it is a binding choice, not a new shape. Install the spec loop and bind its two agent nodes to different vendors: the implementer on one, the reviewer on another.

Everything else stays as the loop defines it. The gate still sits before the reviewer, failures still route back to the implementer, and the reviewer still lands the commit. Only the vendors differ.

For a harder review bar on top of the split, the single-committer graph adds a second, adversarial reviewer behind the same gate. Bind each of its agent roles across vendors the same way.

## Try it

```sh
canopy graph install spec-loop
```

Bind the implementer and the reviewer to different vendors at install time, and keep the review prompt to diff plus spec plus "find what is wrong".
