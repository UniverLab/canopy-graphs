---
title: Resilient members
summary: Cascade and round-robin ensembles give one node a fallback ladder or a rotation across providers, so a quota death is not a failed run.
status: in use
implements:
  - node:resilient-implementer
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

The catalogue's `resilient-implementer` node is a cascade ensemble of
two implementers. Both members share the implementer prompt; each is
an empty object so that nobody's platform or model is baked in. The
cascade keys from `nodes/resilient-implementer/node.json`:

```json
"config": {
  "kind": "cascade",
  "min_pass": 1,
  "members": [
    {},
    {}
  ]
}
```

The shared `prompt_template` is the catalogue's implementer role,
verbatim. `kind: cascade` makes the members a ladder: the first pass
wins, and a quota or harness failure on the first member falls through
to the second. `min_pass: 1` says one passing member is enough. Bind a
different harness to each member at install time, ideally across
providers, so one quota death cannot end the run.

The graph schema allows a third ensemble kind, `round_robin`, in which
members take turns. No catalogue node uses it yet.

For the failure side of the same problem, see the `resilience` node. It diagnoses a failure as quota, glitch or a genuine defect, and routes to the right next step. It is a different tool: it reacts after a node fails, where members prevent the failure from ending the run.

## Try it

```sh
canopy node install resilient-implementer
```
