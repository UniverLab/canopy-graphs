---
title: Adversarial review
summary: Review the work with models of different families, one told to break it, and back the verdict with deterministic checks.
status: standard
implements:
  - graph:single-committer
sources:
  - title: Building effective agents (Anthropic), evaluator-optimizer workflow
    url: https://www.anthropic.com/engineering/building-effective-agents
---

# Adversarial review

## The problem

A model that wrote a change is a poor judge of it, and a model of the same family as the author tends to share its blind spots. A friendly review passes work that a hostile one would reject. On cheap models, where a single pass is unreliable, that gap costs real runs.

## The pattern

Put at least two reviewers behind the implementer. One reads constructively. The other is told to assume the diff is wrong until every line earns its trust. Both must pass.

Run the reviewers on models from different families, so they do not fail in the same way. Then put deterministic checks (formatter, linter, tests) in front of them, so no reviewer is asked to judge what a command can decide.

Anthropic describes the shape in its evaluator-optimizer workflow: "one LLM call generates a response while another provides evaluation and feedback in a loop." The lab's version adds a second, adversarial evaluator and mixes model families.

## What we saw

Early in Canopy, a bug defeated Opus 4.6. The operator spent many tokens on it without a fix. MiMo 2.5 solved it on the first try. The operator no longer remembers which bug it was.

This is one anecdote, and it proves nothing about which model is better. The lesson the lab took from it is that the models are different: MiMo's training may have covered a problem that Opus's did not. A reviewer from another family may see what the author's family cannot.

Since then, the lab's standard is adversarial review by models of different families, plus deterministic checks.

## In Canopy

A parallel ensemble holds the reviewers. Its `min_pass` is 2, so both members must pass. Member 1 uses the shared constructive prompt. Member 2 overrides it with the adversarial one. No member names a platform or model, so whoever installs the graph binds a different family to each.

This excerpt is from `graphs/single-committer/graph.json`, with the prompt fields left out:

```json
{
  "name": "reviewers",
  "kind": "parallel",
  "commit_rights": false,
  "entry_from_node": "full-gates",
  "entry_condition": "pass",
  "on_pass_to": "committer",
  "on_fail_to": "implementer",
  "min_pass": 2,
  "timeout_minutes": 30
}
```

The reviewers have no commit rights. A single committer lands the work only after both pass, and a failure routes back to the implementer.

## Try it

```sh
canopy graph install single-committer
```

Bind a different model family to each reviewer member when you install it.
