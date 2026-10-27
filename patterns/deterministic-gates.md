---
title: Deterministic gates
summary: Shell checks decide pass or fail, never an LLM judge, and a failure carries its reason into the next prompt.
status: standard
implements:
  - graph:spec-loop
  - node:cargo-gates
sources:
  - title: Building effective agents (Anthropic), ground truth from the environment
    url: https://www.anthropic.com/engineering/building-effective-agents
  - title: How we built our multi-agent research system (Anthropic), deterministic safeguards
    url: https://www.anthropic.com/engineering/multi-agent-research-system
---

# Deterministic gates

## The problem

An LLM asked "does this pass?" can be wrong, flattered, or tired. On a cheap model it is also another paid call. Whether the formatter is clean and the tests are green is a fact about the repo, not an opinion.

## The pattern

Let a shell command decide. The gate runs the formatter, the linter and the tests, and its exit code is the verdict. No model sits in the judging seat.

A gate that only says "fail" teaches the next attempt nothing. So the failure's output goes into the next prompt, and the implementer sees why it was sent back.

Anthropic's agent guidance makes the same point about grounding: agents should "gain 'ground truth' from the environment at each step." Its multi-agent write-up also pairs model judgment with deterministic safeguards such as retries and checkpoints.

## What we saw

The lab runs cheap, unreliable models with tight quota. Its standard loop puts the gates before any reviewer, so a reviewer's quota is never spent on code that does not compile.

The lab has not measured how much this saves. It is a design choice, not a benchmark.

## In Canopy

A `check` node holds the command. Edges route `pass` forward and `fail` back to the implementer. In `spec-loop`, the gate and its two edges look like this:

```json
{
  "from_node": "full-gates",
  "to_node": "reviewer",
  "condition": "pass"
},
{
  "from_node": "full-gates",
  "to_node": "implementer",
  "condition": "fail"
}
```

The implementer's preset reads `{{previous_feedback}}` and must address every point in it, which is how a failure's reason reaches the next attempt.

The `cargo-gates` node is the Rust version of the gate, from `nodes/cargo-gates/node.json`:

```json
"config": {
  "command": "cargo fmt --all && cargo clippy --all-targets -- -D warnings && cargo test"
}
```

In `spec-loop`, the gate command is a failing placeholder. Replace it with your project's own commands.

## Try it

```sh
canopy graph install spec-loop
canopy node install cargo-gates
```
