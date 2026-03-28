This session was launched via `workflow-role`, which set `CURRENT_ROLE=workflow` and started Claude Code in a project directory.

## How instructions reach you

1. **Role instructions** — injected by a SessionStart hook. These tell you to read `.workflow/active-role.md` for your specific workflow assignment.
2. **Infrastructure manifest** — injected alongside role instructions.
3. **Project instructions** — loaded natively by Claude Code from CLAUDE.md files in the project tree.

## Workflow skills

Skills like `/workflow-review` and `/workflow-spec` are available but may duplicate your role. Follow your assigned role instructions directly.
