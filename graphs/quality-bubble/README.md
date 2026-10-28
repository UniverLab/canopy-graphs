# quality-bubble

What this graph is for: covering the change already in the worktree
with honest tests, with no spec to write. The bubble tests the current
uncommitted change by design, so it has no lock and no clean-tree
start check — run it with the change already present but uncommitted.
Every failure routes back to the implementer for another attempt.

Nodes in order:

1. `implementer` (agent) — writes a small, focused set of tests for the uncommitted change; never commits.
2. `test-run` (check) — runs the project's tests (installer placeholder).
3. `test-size-cap` (check) — rejects a bloated diff: added plus removed lines must stay within the cap.
4. `anti-cheat-reviewer` (agent, commit rights) — rejects cheating tests (copied expectations, tests that cannot fail, disabled tests, over-mocking) and lands exactly one commit when the tests honestly cover the change.

Fail edges: `test-run`, `test-size-cap`, and `anti-cheat-reviewer`
all route `fail` back to `implementer`.

What the installer must fill in:

- Harness: every agent node carries no `platform`/`model` — bind one per agent node (`implementer`, `anti-cheat-reviewer`) at install time.
- Gate command: `test-run` is a failing placeholder. Replace it with the project's test command.
- Size cap: `test-size-cap` caps the diff at 600 added-plus-removed lines. Adjust the cap to the project's test budget (marked `EDIT ME` in the command).
- Workdir: point the installed graph at the repository it should run in.

Install:

```sh
canopy graph install quality-bubble
```

## How the lab runs it

Coverage instrumentation relinks the whole crate and runs the suite
under a coverage tool. On a small machine that is the kind of job
that takes the machine down rather than merely being slow. But CI
already measures coverage on every pull request, on a runner that
does not care. So do not measure locally. Read the measurement CI
already took, and let a cheap model turn it into tests.

### The shape

A cron trigger fires the graph on a schedule. The first node is
deterministic and spends no tokens: if CI's coverage job is green, it
exits non-zero and the tick ends there. Otherwise a test writer
writes tests from CI's numbers, the gates run them, a committer lands
them, and a final check verifies the commit and pushes, so CI
re-measures without a human in the graph.

### The verdict node

The whole point is that this node is bash, not a model. It answers
one question — is there work? — and, when there is, hands the next
node the actual numbers instead of an invitation to guess.

```bash
set -u
branch=$(git rev-parse --abbrev-ref HEAD)
run=$(gh run list --branch "$branch" --workflow CI --status completed \
      --limit 1 --json databaseId --jq '.[0].databaseId')
[ -n "${run:-}" ] && [ "$run" != "null" ] || { echo "NO-CI"; exit 1; }

cov=$(gh run view "$run" --json jobs \
      --jq '.jobs[]|select(.name|endswith("Coverage"))|.conclusion')
[ "$cov" = "failure" ] || { echo "NOTHING-TO-DO: coverage is $cov"; exit 1; }

job=$(gh run view "$run" --json jobs \
      --jq '.jobs[]|select(.name|endswith("Coverage"))|.databaseId')
log=$(gh api "repos/{owner}/{repo}/actions/jobs/$job/logs" \
      | sed 's/^[0-9TZ:.+-]\{20,\} //')

table=$(printf '%s\n' "$log" \
        | grep -E '^(Filename|TOTAL|[A-Za-z0-9_/.-]+\.rs)[[:space:]]' | tail -70)
if [ -n "$table" ]; then
  echo "--- COVERAGE REPORT FROM CI ---"; printf '%s\n' "$table"
else
  echo "--- NO COVERAGE TABLE: the job died before measuring ---"
  printf '%s\n' "$log" | grep -E 'FAILED|panicked at|^error|test result:' | tail -20
fi
exit 0
```

That last branch is not defensive padding. A coverage job fails for
two very different reasons — the percentage dropped, or the suite
died before any percentage existed — and the writer needs to know
which. Print the failing test when there is no table, and the next
node fixes a test instead of inventing coverage for code that never
ran. The node's stdout arrives at the writer as previous feedback.

### The two rules that make the writer safe

A model asked to raise coverage will, given the chance, change the
code to be easier to test, and will run the coverage tool to see how
it is doing. Both have to be shut down explicitly, in the prompt's
prohibited section:

```
NEVER measure coverage on this machine. Not the project's coverage tool,
not any equivalent. CI measures coverage; you do not. This rule has no
exception and no "just once to check".

NEVER change non-test code. No production behaviour, no signatures, no
visibility, no dependencies. Your diff must contain test code and nothing
else. If a function cannot be tested without changing it, report FAIL naming
the function — a production change smuggled in as a test change is a failed
run, even if the tests pass.
```

The committer enforces the second one by reading the diff rather than
trusting the report, and treats any non-test hunk as an automatic
request-changes. Free models are agreeable; the deterministic check is
what holds.

Worth adding for any suite that runs under nextest: say so in the
prompt. Nextest gives each test its own process, and a suite that
relies on that will accept a test that sets an environment variable —
which then breaks the day someone runs the plain test command.

### Pushing from the graph

The final check verifies the commit landed and pushes it:

```bash
test -z "$(git status --porcelain -- src/)" || exit 1
test -n "{{spec_committed_head}}" || exit 1
test "$(git rev-parse HEAD)" = "{{spec_committed_head}}" || exit 1
branch=$(git rev-parse --abbrev-ref HEAD)
case "$branch" in main|master|develop|HEAD) echo "refusing"; exit 1;; esac
out=$(git push origin "HEAD:$branch" 2>&1); code=$?
printf '%s\n' "$out" | tail -5; exit $code
```

The committed-head marker (not the start-head marker) is what makes
this gate mean "the committer node landed this spec's own commit",
not just "HEAD moved since the spec began". It requires the committer
above to carry commit rights.

Two requirements. The remote must be HTTPS with a credential helper,
and an SSH remote will hang on a passphrase prompt no one is there to
answer. And the branch allowlist is not optional: this is the one
node in the graph whose mistakes leave the machine. If workflow files
are ever part of what gets pushed, the token needs the workflow scope
on top of repo, or the push is rejected naming the workflow file and
nothing else.

### What will bite you

- **A completed spec never re-runs.** Selection takes running first,
  then pending or interrupted; completed and failed are skipped, and
  a launch with no eligible spec is an error, not a quiet no-op. A
  cron graph over a single standing "raise coverage" spec works
  exactly once. Give it a queue of specs, one per module, and let the
  trigger walk the queue.
- **Each cycle costs a full CI run.** The graph pushes, CI
  re-measures, the next tick reads the new verdict. Set the cron
  interval above the CI round-trip or ticks keep reading a stale
  verdict.
- **Coverage percentage is not the goal, and a model optimizing it
  will say it is.** Tests that execute a line without asserting on
  the result move the number and catch nothing. Say so in the
  writer's prompt and reject them in the committer's.
