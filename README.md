# canopy-graphs

The official [Canopy](https://univerlab.org/canopy) graphs and nodes, listed on
[graphs.univerlab.org](https://graphs.univerlab.org).

Official documents are ordinary catalogue documents: this repo has the same
layout any community repo uses. The only difference is the owner. Fork it, or
press **Use this template**, to start your own.

## What is here

| Graph | What it is for |
|---|---|
| [canopy-loop](graphs/canopy-loop) | The lab's loop as it runs: locked repo, designer, implementer, full gates, reviewer, commit, auditor, gates again, docs, unlock. |
| [canopy-loop-plain](graphs/canopy-loop-plain) | The same roles as plain nodes: no ensembles, no lock. The starting point. |

| Node | What it is for |
|---|---|
| [designer](nodes/designer) | Investigates the spec against the code and saves the plan as its spec note. |
| [implementer](nodes/implementer) | The implementer step of a spec loop. |
| [reviewer](nodes/reviewer) | The completeness review: locates every requirement and asks for changes. |
| [auditor](nodes/auditor) | Reviews against the spec, repairs what it objects to, never commits. |
| [full-gates-cargo](nodes/full-gates-cargo) | The Rust full gate: formatter, linter and tests in one check command. |
| [full-gates-web](nodes/full-gates-web) | The TypeScript/web full gate: lint, typecheck, tests, build, end-to-end. |
| [documenter](nodes/documenter) | Writes the user docs the change needs and leaves them uncommitted. |

Install an official document by its name:

```sh
canopy graph install canopy-loop
canopy node install implementer
```

## Patterns

The [patterns](patterns) are short pages on the orchestration ideas behind these
documents: where each comes from, what the lab saw, and which document installs it.

## Make your own graphs repo

1. **Create a public repo.** Fork this one or use it as a template. Delete the
   documents you do not want to share.
2. **Lay it out like this one.** A graph lives at `graphs/<slug>/graph.json`
   and a node at `nodes/<slug>/node.json`. The `<slug>` equals the document's
   `name`. Each folder carries a `README.md` that says what the document does.
3. **Keep documents harness-free.** Shared documents never carry `platform`,
   `model` or `cli`. Whoever installs a document chooses the harnesses.
4. **Check before you push:**
   ```sh
   canopy graph validate graphs/<slug>/graph.json
   ```
5. **Add the `canopy-graph` topic** under Settings → About → Topics. The
   catalogue indexes it within minutes.

People then install your documents with `owner/repo/slug`:

```sh
canopy graph install <owner>/<repo>/<slug>
```

The [Publish page](https://graphs.univerlab.org/publish) gives the full rules.

## Proposing an official document

Open a pull request that adds the document folder.

## blocked.json

`blocked.json` lists the documents the catalogue hides after a report. Reports
come in through the **Report** link on each document page. This file
matters only in this repo; a fork can leave it empty.

## License

Apache-2.0. See [LICENSE](LICENSE).
