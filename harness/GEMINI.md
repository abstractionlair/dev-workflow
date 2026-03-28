This session was launched via `HARNESS=gemini workflow-role`, which set `CURRENT_ROLE=workflow` and started Gemini CLI in a project directory.

## How instructions reach you

1. **Role instructions** — injected by a BeforeAgent hook. These tell you to read `.workflow/active-role.md` for your specific workflow assignment.
2. **Infrastructure manifest** — injected alongside role instructions.
3. **Project instructions** — loaded natively by Gemini CLI from GEMINI.md files in the project tree.
