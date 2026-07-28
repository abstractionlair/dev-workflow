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
