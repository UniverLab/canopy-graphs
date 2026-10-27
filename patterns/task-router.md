---
title: Task router
summary: An experiment, a router judges the task by kind or complexity and sends it to a model suited to it.
status: experiment
implements: []
sources:
  - title: Building effective agents (Anthropic), routing
    url: https://www.anthropic.com/engineering/building-effective-agents
---

# Task router

This pattern is an experiment. The lab has not tested it properly.

## The problem

One model for every task wastes money on easy work and risks quality on hard work. Some tasks also need a quality that cheap models lack, such as visual taste.

## The pattern

Insert a router in front of the work. It reads the task, judges its kind or its complexity, and sends it down the branch that fits.

Anthropic names this workflow: "Routing classifies an input and directs it to a specialized followup task." It is a standard pattern.

## What we saw

Nothing yet, and that is the point of the label. The router is an untested hypothesis in Canopy, not an observed result.

The idea is an example, not a finding. Graphic design and assets would go to a model with strong taste, and everything else to cheap models. Opus 5.5 designed the univerlab.org site and its assets with the operator, which is why the hypothesis looks plausible to the lab. One project is not evidence that a router would choose well, or that it would cost less than a single model.

Risks worth testing: the router can misjudge, and every routing decision is itself a model call.

## In Canopy

The graph schema has a `router` node kind, and edges can carry a route label as their condition. A router graph would send each label to a different branch.

No document in this catalogue uses a router yet, and this repository has no real file to copy an excerpt from. This page therefore implements no catalogue document.

## Try it

No catalogue document implements this pattern yet. If you build one, the repo README explains how to publish it.
