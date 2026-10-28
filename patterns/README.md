# Patterns

Evergreen pages on the agent-orchestration patterns the lab runs on cheap,
unreliable models with tight quota. Each page names its primary source, tells
what the lab saw, shows how a Canopy graph expresses the pattern, and ends with
the catalogue document you can install.

| Pattern | Status | Summary |
|---|---|---|
| [Adversarial review](adversarial-review.md) | standard | Review the work with models of different families, one told to break it, and back the verdict with deterministic checks. |
| [Deterministic gates](deterministic-gates.md) | standard | Shell checks decide pass or fail, never an LLM judge, and a failure carries its reason into the next prompt. |
| [Cross-platform check](cross-platform-check.md) | in use | The reviewer runs on a different vendor from the implementer, so one family's blind spots and one quota ceiling cannot stall the graph. |
| [Reviewer fixes](reviewer-fixes.md) | in use | The reviewer who rejects the work fixes it, instead of writing change requests for another model to read. |
| [Review panel](review-panel.md) | experiment | Each reviewer owns one axis, security, performance, maintainability, or project patterns, and every axis must pass. |
| [Parallel proposers](parallel-proposers.md) | in use | Several cheap models draft the same spec in parallel, then one implementer builds on the best ideas. |
| [Resilient members](resilient-members.md) | in use | Cascade and round-robin ensembles give one node a fallback ladder or a rotation across providers, so a quota death is not a failed run. |
| [Deterministic commit](deterministic-commit.md) | in use | A check node commits with the spec's own Conventional Commit subject, and a model never writes the commit. |
| [Event intake](event-intake.md) | in use | A graph that fires on a watched file, turning outside events into specs while nobody is watching. |
| [Task router](task-router.md) | experiment | An experiment: a router judges the task by kind or complexity and sends it to a model suited to it. |

`standard` is the lab's default practice, `in use` runs in the lab but is not
yet the default, and `experiment` is untested.

## Choosing a shape

| If the thing you fear is… | Use |
|---|---|
| Broken code landing | Deterministic gates |
| One vendor's blind spot | Cross-platform check |
| A quota ending overnight | Resilient members |
| Spending the good model on exploration | Parallel proposers |
| Spending the good model on trivia | Task router |
| Paying at all before the free tiers are gone | Resilient members (cascade) |
| A whole dimension going unreviewed | Review panel |
| Work arriving while you sleep | Event intake |
| A fix that is right but fragile | Adversarial review |

Shapes compose. The graph that drains the lab's own backlog is gates plus a
resilience branch, and grows a router when routing earns its keep.

## What the engine does not have yet

Collected from the patterns above, because a limitation stated once in
context is worth more than a roadmap:

- **Memory of a spent harness.** Nothing survives a spec boundary to say a
  harness is exhausted until some hour, so a cascade rediscovers it every
  spec. See Resilient members.
- **Severity in a quorum.** Members vote pass or fail; there is no way to say
  "this finding blocks, that one is a note". See Review panel.
- **The triggering event's payload.** A watch-fired graph gets the fact, not
  the path. See Event intake.
