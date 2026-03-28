---
name: workflow-status
description: Show the current state of the dev-workflow in this project. Displays specs, bugs, and reviews with their states, and suggests the next action. Use to see where things stand.
---

# Workflow Status

Show the current workflow state and suggest next actions.

## Process

### 1. Load preamble

```
Read ~/deploy/dev-workflow/preamble.md
```

Follow the state detection instructions in the preamble.

### 2. Scan project state

**Check initialization:**
- Does `.workflow/config.json` exist? If not: "Project not initialized. Run `/workflow-init`." and stop.
- Read `.workflow/config.json` for active stages.

**Scan specs:**
- List `specs/proposed/` — drafts awaiting review
- List `specs/todo/` — approved, ready to implement
- List `specs/doing/` — in progress
- List `specs/done/` — completed

**Scan bugs:**
- List `bugs/to_fix/` — pending
- List `bugs/fixing/` — in progress
- List `bugs/fixed/` — resolved

**Scan strategic docs** (if active):
- Check existence and version of `docs/VISION.md`, `docs/SCOPE.md`, `docs/ROADMAP.md`

**Scan RFCs** (if rfcs/ exists):
- List `rfcs/open/` — pending change requests

### 3. Report state

Present a compact summary:

```
## Workflow Status: [project-name]

### Specs
- Proposed (awaiting review): [list or "none"]
- Approved (ready to build): [list or "none"]
- In progress: [list or "none"]
- Complete: [count]

### Bugs
- To fix: [list or "none"]
- Fixing: [list or "none"]
- Fixed: [count]

### Strategic Docs
- Vision: [exists/missing/skipped]
- Scope: [exists/missing/skipped]
- Roadmap: [exists/missing/skipped]
```

### 4. Suggest next action

Based on state, recommend what to do next:

| State | Suggestion |
|-------|------------|
| No strategic docs and planning is active | `/workflow-plan` to define vision/scope |
| Specs in proposed/ | `/workflow-review` to review pending specs |
| Specs in todo/ | `/workflow-build` to start implementing |
| Specs in doing/ but no recent activity | Check feature branch status |
| No specs anywhere | `/workflow-spec` to specify first feature |
| Bugs in to_fix/ | Investigate and fix pending bugs |
| Open RFCs | Review and decide on change requests |

Report status: **DONE**
