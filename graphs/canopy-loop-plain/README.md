# canopy-loop-plain

What this graph is for: the same roles as [canopy-loop](../canopy-loop),
as plain nodes you can read in one glance. Designer, implementer, full
gates, a completeness reviewer, a reviewer of record that repairs what it
finds, the gates again, a documenter, and one deterministic commit. No
ensembles, no repository lock, no resilience node.

**Start here.** This is the document to copy when you build your own loop:
every role is one node with one prompt, so a change is a change in one
place. Once the shape is yours, `canopy-loop` is the hardened version — it
adds round-robin ensembles so a quota death does not fail the spec, the
gitkit lock so nothing else commits while it runs, a plan check, and a
resilience node that can schedule its own wake-up.

Differences that matter when you move from here to `canopy-loop`:

- **One commit, at the end.** The documenter leaves its docs uncommitted
  and `Commit` lands the spec's work and its docs together, under the
  spec's own message. `canopy-loop` commits the code first, reviews the
  commit, and commits the review fixes separately.
- **The reviewer of record reads the working tree.** Here the change is
  uncommitted when it reviews; in `canopy-loop` it reads `HEAD`.
- **No lock.** Nothing stops another process committing while the loop
  runs, so do not run two of these against the same branch.
- **A designer failure ends the spec.** There is no resilience node to
  triage it and no plan check to bounce it.

Nodes in order:

1. `Start` (check) — a no-op entry node.
2. `Designer` (agent) — investigates and saves the plan as its spec note. The note is the deliverable.
3. `Implementer` (agent) — implements the plan, leaves the tree uncommitted.
4. `Full gates` (check) — `<GATE-SCRIPT>`, holding the `cpu` mutex.
5. `Reviewer 1 completeness` (agent) — locates every requirement at `file:line`. It does not fix and does not commit.
6. `Reviewer 2 review and fix` (agent) — the reviewer of record: repairs what it objects to in the working tree, runs the gate script once, never commits.
7. `Full gates after reviewer 2` (check) — the same gate script over the repairs.
8. `Documenter` (agent) — writes the user docs the change needs and leaves them uncommitted.
9. `Commit` (check, commit rights) — commits everything with `$CANOPY_SPEC_COMMIT_MESSAGE`, the spec's own message.

Fail edges:

- `Full gates` fail → `Implementer`
- `Reviewer 1 completeness` fail → `Implementer`
- `Reviewer 2 review and fix` fail → `Implementer`
- `Full gates after reviewer 2` fail → `Reviewer 2 review and fix`

What to set before installing:

- **The gate script.** `<GATE-SCRIPT>` is the command of both gate nodes
  and is quoted in the implementer's and the second reviewer's prompt.
  Replace it with the command that runs the project's gates. Until you do,
  the gate fails, which is the point.
- **Harnesses.** Every agent node carries no `platform` or `model`; bind
  one to each of the five at install time.
- **The push.** None. The commit stays local, as the `spec-commit` node
  does; add a push to `Commit` if the graph should publish the branch.

Install:

```sh
canopy graph install canopy-loop-plain
```
