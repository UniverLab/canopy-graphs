# spec-loop

What this graph is for: running one spec through the lab's proven
loop — implement, deterministic gates, review-and-commit, proof the
commit landed, release the repo lock, and a human verdict. Every
failure routes back to the implementer for another attempt.

Nodes in order:

1. `start-check` (check) — refuses a dirty worktree before anything runs.
2. `lock-repo` (check) — takes the repo lock for this spec.
3. `implementer` (agent) — implements exactly the spec; never commits.
4. `full-gates` (check) — the deterministic gates (installer placeholder).
5. `reviewer` (agent, commit rights) — reviews the diff against the spec and lands exactly one commit.
6. `commit-check` (check) — proves a commit landed since the spec started.
7. `unlock` (check) — releases the repo lock.
8. `human-verdict` (check) — waits up to 24h for a human verdict file; anything but approval routes back to the implementer.

Fail edges: `full-gates`, `reviewer`, `commit-check`, and
`human-verdict` all route `fail` back to `implementer`.

What the installer must fill in:

- Harness: every agent node carries no `platform`/`model` — bind one per agent node (`implementer`, `reviewer`) at install time.
- Gate command: `full-gates` is a failing placeholder. Replace it with the project's deterministic gates (formatter, linter, tests).
- Workdir: point the installed graph at the repository it should run in.
- Verdict file: a human approves by writing a line containing `VERDICT: APPROVED` to `.canopy/verdict` in the workdir; any other content (e.g. `VERDICT: REJECTED` plus notes) sends the spec back to the implementer.

The commit check uses the `{{spec_start_head}}` form. For the stronger
"this run's own committer landed a commit" check — immune to a
concurrent commit from outside the run — switch it to:

```sh
test -n "{{spec_committed_head}}" && test "$(git rev-parse HEAD)" = "{{spec_committed_head}}"
```

Install:

```sh
canopy graph install spec-loop
```

## How the lab runs it

### The spec is the real input

A graph is only as good as what it is fed. The most common cause of a
disappointing run is not the topology but a spec that left a decision
to the implementer. A spec that says "choose an appropriate mechanism
and document why" delegates architecture to a model tuned for
throughput. Decide first, then write the spec with the decision
already in it and the reasoning attached:

```
DECISIONS ALREADY MADE - DO NOT RE-LITIGATE
1. Back up with VACUUM INTO, not a file copy. Copying a live SQLite with a
   WAL can produce a broken copy even when the original is healthy.
2. Exactly one backup, verified immediately after it is written. N backups
   force a choice of which to restore, which is the decision least fit to
   make mid-incident.
```

A section named IN SCOPE / OUT OF SCOPE is worth more than any prompt
engineering downstream of it.

### Draining a bug backlog

This graph built most of Canopy: twenty small specs and a night. Four
roles, six edges: implement passes to gates, gates pass to review,
implement fails to resilience, gates and review fail back to
implement, resilience passes back to implement. Review on pass has no
edge: that is how a spec completes. Resilience on fail has none
either: the spec fails honestly, having already scheduled the graph's
own wake-up.

Keep the graph (the machine) separate from the queue of specs (the
work); the same graph points at a different backlog tomorrow.

The lab binds implement to its strongest model with a long timeout,
review to a cheap model with commit rights, and resilience to another
cheap model on a different quota from implement.

The implementer's prompt, in the tagged-section style:

```
# [ROLE]
You implement one spec in an existing codebase. You are not the
architect: every design decision you need has already been made and is in
the spec.

# [SPEC]
{{spec_content}}

# [PREVIOUS FEEDBACK]
{{previous_feedback}}

# [HOW]
- Read before you write. Match the surrounding code's naming and idiom.
- You have NO COMMIT RIGHTS. Leave your work uncommitted; a later node
  reviews the diff and commits it.
- Run the project's gates yourself before reporting pass.

# [REPORTING]
Call graph_complete_node with status "pass" only if the spec is fully
implemented. Partial work is "fail" with a description of what is missing.
```

The reviewer's prompt, which carries the load this shape exists for:

```
# [ROLE]
You review one diff against one spec, and you commit it if it holds up.

# [WHAT]
The gates already passed. That means nothing is broken. It says nothing
about whether the spec was HONORED, and that distinction is your entire
job.

# [HOW]
- Write a verdict for EVERY requirement in the spec, one line each, BEFORE
  you decide. A reviewer that must enumerate is a reviewer that must look.
- A missing required test is CHANGES_NEEDED even if the code is perfect.
- When in doubt, CHANGES_NEEDED. A bounce costs minutes; a falsely-closed
  spec costs days.
- Do not approve out of courtesy, for effort spent, or because it almost
  meets the spec.
- Stage only the files this spec touched. `git add -A` sweeps a previous
  spec's leftovers into this commit.
- Conventional Commits format. No trailers.
- Prove it: run `git log -1 --oneline` and include the output. A reviewer
  once reported a commit that did not exist.

# [SPEC]
{{spec_content}}
```

The resilience prompt, kept short for a cheap model doing one narrow
job:

```
# [ROLE]
The previous node's CLI died. Decide whether it can be retried.

# [HOW]
1. Read the failure output below. Do not open files, do not run git.
2. If it names a usage, session or quota limit: schedule the graph's own
   wake-up with the failure text copied verbatim as the reset message. Do
   NOT compute a time yourself — the engine derives it from the message.
   Then report FAIL.
3. If the engine answers that it could not derive a reset instant: report
   a blocker describing which limit was hit and what a human must do.
   That call finalizes the run; do not also report.
4. If it is a transient failure you can repair: repair it, report PASS.
5. Anything else, or any doubt: report FAIL, summary "UNDIAGNOSED".

# [OUTPUT]
{{previous_feedback}}
```

Step 3 exists because of a real run. The implementer died on a monthly
spend limit, which names no reset time because a spend ceiling is not
a rolling window. The classifier passed the message on, the engine
could not derive an instant, and with no third branch the graph
stopped with its reason readable only in the database. A blocker
report is the difference between a graph that is failed and one that
is blocked with a sentence saying what to do. Prefer it for every
failure that will not lift on its own.

Note what the prompt never does: it never computes a reset time.
Models do not know what time it is, and a real run that was allowed
the arithmetic scheduled its wake-up a day late. Let the engine parse
the message; the model only decides which message it is looking at.

If the graph grows a designer node ahead of implement, have it write
the plan with spec_note_write rather than a file in the repo: later
prompts already carry every note, and a verify check reads the notes
file for the plan's heading.

### Running it

Point the graph at the queue and run. It drains pending specs in
order; completed specs are skipped on relaunch, so recovery after a
crash is just running it again.

### What will bite you

- **`prompt`, not `prompt_template`.** The engine reads
  `prompt_template`, then `prompt_preset`, then a default. There is no
  `prompt` key. A graph configured with `prompt` runs silently on the
  fallback: eight specs here ran with no instructions at all before
  anyone noticed, because the fallback is plausible and the results
  look like a model ignoring you.
- **`always` where `pass` was meant.** Routing a failed implement
  into the reviewer approves work never written. It is the single most
  expensive edge mistake.
- **A resilience node on the implementer's quota.** It will be dead
  exactly when it is needed.

### Cross-cutting notes

- **A cycle with no entry node.** The retry shape is a cycle where
  every node has an incoming edge. The engine resolves a spec's start
  node as the one with no incoming edge, falling back to the lowest
  position. Keep the intended start at position 1 and treat position
  as load-bearing.
- **Duplicate outgoing edges.** Two edges leaving one node, both
  matching, pointing at different targets aborts the run after the
  node's work is done. Dedupe by (from_node, condition) before adding
  them.
- **The CLI resolves from the daemon's environment, not your shell.**
  A binary on your PATH but not the daemon's fails to spawn mid-run.
  Use an absolute binary path in the harness config.
- **Small specs are a quota strategy.** A big spec means a long run
  means a burnt ceiling means a failed spec. Specs that finish in
  under half an hour are the difference between losing one spec and
  losing a night.
- **Queue order can split a group.** A reorder that would separate
  one context group's pending specs is refused and nothing changes.
  Grouped specs share a warm session with their siblings.
- **Commit rights are configuration, not instruction.** Three
  different models have committed against a capitalized "you have no
  commit rights" rule, each time handing reviewers an empty diff.
  Mark the committer in its node config and let the engine enforce
  it.
