# Patterns

Evergreen pages on the agent-orchestration patterns the lab runs on cheap,
unreliable models with tight quota. Each page names its primary source, tells
what the lab saw, shows how a Canopy graph expresses the pattern, and ends with
the catalogue document you can install.

| Pattern | Status | Summary |
|---|---|---|
| [Adversarial review](adversarial-review.md) | standard | Review the work with models of different families, one told to break it, and back the verdict with deterministic checks. |
| [Deterministic gates](deterministic-gates.md) | standard | Shell checks decide pass or fail, never an LLM judge, and a failure carries its reason into the next prompt. |
| [Reviewer fixes](reviewer-fixes.md) | in use | The reviewer who rejects the work fixes it, instead of writing change requests for another model to read. |
| [Parallel proposers](parallel-proposers.md) | in use | Several cheap models draft the same spec in parallel, then one implementer builds on the best ideas. |
| [Resilient members](resilient-members.md) | in use | Cascade and round-robin ensembles give one node a fallback ladder or a rotation across providers, so a quota death is not a failed run. |
| [Deterministic commit](deterministic-commit.md) | in use | A check node commits with the spec's own Conventional Commit subject, and a model never writes the commit. |
| [Task router](task-router.md) | experiment | An experiment: a router judges the task by kind or complexity and sends it to a model suited to it. |

`standard` is the lab's default practice, `in use` runs in the lab but is not
yet the default, and `experiment` is untested.
