---
name: workflow-spec
description: Write a feature specification — Socratic dialogue then structured spec writing. Use when specifying a new feature or refining an existing spec.
---

# Feature Specification

Write a feature spec using the dev-workflow process. Two modes: **explore** (Socratic dialogue to clarify thinking) and **write** (produce the spec document).

## Process

### 1. Load context

```
Read ~/deploy/dev-workflow/preamble.md
```

Follow the preamble instructions: scan project state, check config.

### 2. Determine mode

If the user has a clear feature in mind and wants to write directly, go to **Write** mode.
If the user wants to explore and clarify their thinking first, go to **Explore** mode.
If unclear, ask: "Do you want to think this through first, or are you ready to write the spec?"

### 3. Load spec-writer role

```
Read ~/deploy/dev-workflow/roles/spec-writer.md
Read ~/deploy/dev-workflow/schemas/spec.md
```

Follow the spec-writer role instructions. The role covers both exploration (Socratic dialogue about interfaces, edge cases, acceptance criteria) and writing (producing the spec) in one flow. Start with exploration if the user needs to clarify, or go straight to writing if they're ready.

Key behaviors:
- Ask probing questions about interfaces, edge cases, error conditions
- Challenge assumptions: "What happens when...?"
- Draw out acceptance criteria through concrete examples
- Read existing context (vision, scope, roadmap, system map, guidelines — whatever exists)
- Write the spec following the schema structure
- Place the spec in `specs/proposed/<feature-name>.md`

### 5. After writing

- Suggest review: "This spec is in `specs/proposed/`. Run `/workflow-review` to have it reviewed, or open a different model terminal for cross-model review."
- If the model-config suggests a different model for review, mention it.

Report status: **DONE** with path to the created spec.
