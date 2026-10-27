---
title: Resilient members
summary: Cascade and round-robin ensembles give one node a fallback ladder or a rotation across providers, so a quota death is not a failed run.
status: in use
implements: []
sources:
  - title: How we built our multi-agent research system (Anthropic), resilience
    url: https://www.anthropic.com/engineering/multi-agent-research-system
---

# Resilient members

## The problem

Cheap models run out of quota, and harnesses fail mid-run. A node bound to a single provider turns either event into a failed run, even though the work itself was fine.

## The pattern

Give a node several members instead of one.

- **Cascade:** the members form a ladder. If the first fails on quota or harness, the run falls through to the next.
- **Round robin:** the members take turns, so spend rotates across providers and no single quota is drained first.

Anthropic describes the same concern in its multi-agent write-up: restarts are costly, so the team "built systems that can resume from where the agent was when the errors occurred." Letting a node fall to another member is a cheaper cousin of that idea.

## What we saw

Cascade and round robin were not invented for resilience. They reuse the ensemble idea from the parallel proposers: an ensemble already holds several members for one node, so the same structure can hold fallbacks instead of parallel drafts.

The lab has no measurements on how many runs this saved, and this page does not claim any.

## In Canopy

The graph schema allows three ensemble kinds: `parallel`, `cascade` and `round_robin`. A cascade or round-robin ensemble is a graph ensemble with that `kind` and a list of members, each of which can name its own platform and model.

No document in this catalogue uses a cascade or round-robin ensemble yet, and the node schema does not show how to ship one harness-free as a node. For that reason this page implements no catalogue document, and carries no excerpt: the repository has no real file to copy one from.

For the failure side of the same problem, see the `resilience` node. It diagnoses a failure as quota, glitch or a genuine defect, and routes to the right next step. It is a different tool: it reacts after a node fails, where members prevent the failure from ending the run.

## Try it

No catalogue document implements this pattern yet. When one exists, its install command will appear here.
