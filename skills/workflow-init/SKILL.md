---
name: workflow-init
description: Initialize the dev-workflow structure in the current project. Sets up directories for specs, bugs, reviews, and workflow config. Use when starting a new project or adding structured workflow to an existing one.
---

# Initialize Dev-Workflow

Set up the dev-workflow directory structure in the current project.

## Process

### 1. Check for existing workflow

Look for `.workflow/config.json` in the current project. If it exists, report current state instead of re-initializing. Suggest `/workflow-status` for details.

### 2. Determine project scope

Ask the user one question: **Is this a large project that needs strategic planning (vision, scope, roadmap), or should we go straight to feature specs?**

- **Large**: `active_stages` includes all stages, `skip_stages` is empty
- **Standard** (default): `active_stages` is `["spec", "skeleton", "tests", "implementation"]`, `skip_stages` is `["vision", "scope", "roadmap"]`

### 3. Create directory structure

Read the ontology for the canonical layout:
```
Read ~/deploy/dev-workflow/ontology.md
```

Then create directories based on active stages:

**Always create:**
```
.workflow/
specs/proposed/
specs/todo/
specs/doing/
specs/done/
bugs/to_fix/
bugs/fixing/
bugs/fixed/
reviews/specs/
reviews/implementations/
reviews/bugs/
```

**If strategic planning is active, also create:**
```
docs/
rfcs/open/
rfcs/approved/
rfcs/rejected/
```

Add `.gitkeep` files to empty directories so git tracks them.

### 4. Write config

Copy `~/deploy/dev-workflow/templates/workflow-config.json` to `.workflow/config.json`. Fill in `project_name` from the directory name or ask the user. Set `active_stages` and `skip_stages` based on step 2.

### 5. If strategic planning is active

Create stub files:
- `docs/VISION.md` with header template from the vision schema
- `docs/SCOPE.md` with header template from the scope schema
- `docs/ROADMAP.md` with header template from the roadmap schema

### 6. Report

Tell the user what was created and suggest next steps:
- If strategic planning active: "Start with `/workflow-plan` to define vision and scope"
- If standard: "Start with `/workflow-spec` to specify your first feature"

Report status: **DONE**
