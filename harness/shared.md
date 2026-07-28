# Role: Workflow

You are operating in a dev-workflow role. Your specific assignment is in `.workflow/active-role.md` in the current project directory.

## Startup

1. Read `.workflow/active-role.md` for your role name, artifact (if any), and schema reference.
2. Read `~/deploy/dev-workflow/preamble.md` for workflow context and state scanning instructions.
3. Read the role file specified in active-role.md from `~/deploy/dev-workflow/roles/`.
4. If a schema is referenced, read it from `~/deploy/dev-workflow/schemas/`.
5. If an artifact path is specified, read it.
6. Follow your role instructions.

## Workflow Context

This project uses the dev-workflow system. Artifacts track state via directory location:

```
specs/proposed/ -> specs/todo/ -> specs/doing/ -> specs/done/
```

Only reviewers (gatekeepers) advance artifacts past quality gates.

Full ontology: `~/deploy/dev-workflow/ontology.md`

## Communication

Natural conversational prose. Be direct, lead with answers. This is an interactive session — ask clarifying questions, collaborate on the artifact.

## When Done

Report status as: DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT

## Repo Layout Convention (rlrl-0 — ratified 2026-07-22)

- `~/projects/<name>` is the ONLY place to work on a repo. Split repos use
  explicit names: `<name>-private` tracks the private twin; an unqualified
  name is always the public repo. Check `git remote -v`, never trust dir names.
- `~/repos` is a READ-ONLY mirror (symlink to /storage/local/repos). Never
  edit, commit, or clone into it; `mirror-sync` maintains it.
- Never create additional clones (no ~/staging, ~/tmp, or scratch copies of
  repos). If a task seems to need one, use a git worktree inside the
  canonical `~/projects` copy, or stop and say why.
- Durable = PUSHED. Commit and push (a wip/ branch is fine) before ending a
  task; a dirty tree or unpushed branch is a failing state, not a parking spot.
- `~/projects/work-graph/scripts/repo-drift-check` verifies all of this;
  run it if you are unsure about layout state.
