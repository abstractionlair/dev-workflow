---
name: workflow-review
description: Review any workflow artifact — macro (design/compliance) or micro (code quality). Gatekeeper for state advancement. Use when an artifact needs review before proceeding.
---

# Workflow Review

Review a workflow artifact. Two review types:

- **Macro review**: Big picture — design quality for docs, spec fidelity for code. Owns gatekeeper actions.
- **Micro review**: Line-level code quality, bugs, naming, style. Code artifacts only. No gatekeeper actions.

## Process

### 1. Load context

```
Read ~/deploy/dev-workflow/preamble.md
```

Follow the preamble instructions to scan project state.

### 2. Determine review type

If the user specified (e.g., "macro review", "micro review", "code review"), use that.

Otherwise, infer:
- Reviewing docs (vision, scope, roadmap, spec) → always **macro**
- Reviewing code → default **macro** (checks against spec), suggest micro as follow-up

### 3. Load the reviewer role

**For macro review:**
```
Read ~/deploy/dev-workflow/roles/macro-reviewer.md
```

**For micro review:**
```
Read ~/deploy/dev-workflow/roles/micro-reviewer.md
```

### 4. Identify artifact and load schema

If the user specified an artifact path, use that. Otherwise, scan for reviewable artifacts:
- `specs/proposed/` — specs awaiting review
- Check for any artifacts the user mentioned

| Artifact location | Schema to load |
|-------------------|----------------|
| `specs/proposed/` | `~/deploy/dev-workflow/schemas/spec.md` |
| `docs/VISION.md` | `~/deploy/dev-workflow/schemas/vision.md` |
| `docs/SCOPE.md` | `~/deploy/dev-workflow/schemas/scope.md` |
| `docs/ROADMAP.md` | `~/deploy/dev-workflow/schemas/roadmap.md` |
| Implementation code | Read the spec it implements |
| `bugs/fixing/` | `~/deploy/dev-workflow/schemas/bug-report.md` |

### 5. Perform review and produce review document

Follow the reviewer role instructions. Write the review to `reviews/<type>/`.

### 6. Gatekeeper action (macro review only, if approved)

Macro-reviewer owns state transitions. Ask user confirmation before executing:

| Artifact | Transition |
|----------|------------|
| Spec in proposed/ | `git mv specs/proposed/<name>.md specs/todo/<name>.md` |
| Implementation (all tests pass) | `git mv specs/doing/<name>.md specs/done/<name>.md` |
| Bug fix (sentinel test added) | `git mv bugs/fixing/<name>.md bugs/fixed/<name>.md` |

Micro-reviewer does NOT perform gatekeeper actions.

### 7. Multi-model note

If reviewing in the same model that wrote the artifact, flag it. For cross-model review, use `workflow-role` in a separate terminal:

```bash
workflow-role macro-reviewer <project> <artifact>   # gemini for docs, opencode for code
workflow-role micro-reviewer <project> <artifact>   # opencode (open models)
```

Report status: **DONE** with review type, verdict, and any state transitions.
