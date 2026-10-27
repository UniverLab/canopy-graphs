---
title: Parallel proposers
summary: Several cheap models draft the same spec in parallel, then one implementer builds on the best ideas.
status: in use
implements:
  - node:ensemble-proposers
sources:
  - title: Mixture-of-Agents Enhances Large Language Model Capabilities (Wang et al.)
    url: https://arxiv.org/abs/2406.04692
  - title: Fusion Router (OpenRouter documentation)
    url: https://openrouter.ai/docs/guides/routing/routers/fusion-router
  - title: Building effective agents (Anthropic), parallelization
    url: https://www.anthropic.com/engineering/building-effective-agents
---

# Parallel proposers

## The problem

One cheap model, one attempt, one set of blind spots. Spending a strong model on every spec is not affordable under tight quota, and a blank spec is the hardest thing to hand to a weak implementer.

## The pattern

Ask several models for a draft of the same spec at the same time. Then give one implementer the drafts and let it build on the best ideas. The implementer never faces the blank spec.

The idea has published precedent. The Mixture-of-Agents paper by Wang et al. arranges several LLMs in layers, where each agent uses the previous layer's outputs as additional context. Anthropic lists parallelization as a workflow: "running the same task multiple times to get diverse outputs."

## What we saw

The lab took the idea from OpenRouter's fusion mode. OpenRouter documents it as a panel of models answering the same prompt in parallel, with an analyst model comparing the answers. It states that the analyst "doesn't merge them." The lab's version is simpler: the drafts go straight to an implementer.

With a similar technique, the operator got reasoning from cheaper models comparable to Fable 5. This is an observation from the lab's own use. It is not a benchmark, and the lab has no controlled comparison.

Parallel drafts cost more calls. OpenRouter estimates a three-model panel at roughly four to five times a single completion, and the lab does not claim the trade is free.

## In Canopy

A parallel ensemble node holds the proposers. All members share one prompt template, and each member is an empty object so that nobody's platform or model is baked in. This is `nodes/ensemble-proposers/node.json`:

```json
{
  "format_version": 1,
  "name": "ensemble-proposers",
  "kind": "ensemble",
  "description": "Three parallel proposers draft concrete solutions for the same spec so a downstream implementer builds from a pre-digested draft.",
  "config": {
    "prompt_template": "Draft a complete solution for this spec. Be concrete and specific — an implementer downstream may build directly from your draft without ever seeing this spec itself, so leave nothing implicit:\n\n{{spec_content}}\n\nPrevious feedback (if any): {{previous_feedback}}",
    "members": [
      {},
      {},
      {}
    ]
  }
}
```

`min_pass` is omitted, so every member must pass. Bind a different model to each member at install time.

## Try it

```sh
canopy node install ensemble-proposers
```
