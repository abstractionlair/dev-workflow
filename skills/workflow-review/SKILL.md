---
name: workflow-review
description: Review any workflow artifact — specs, implementations, bugs. Gatekeeper for state advancement. Use when an artifact needs review before proceeding.
---

# Workflow Review

Review a workflow artifact and optionally advance its state.

## Process

### 1. Load context

```
Read ~/deploy/dev-workflow/preamble.md
Read ~/deploy/dev-workflow/roles/reviewer.md
```

Follow the preamble instructions to scan project state.

### 2. Identify what to review

If the user specified an artifact path, use that.

Otherwise, scan for reviewable artifacts:
- `specs/proposed/` — specs awaiting review
- Check for any artifacts the user mentioned need review

If nothing is pending, report: "No artifacts pending review." and suggest `/workflow-spec` or `/workflow-build`.

### 3. Determine artifact type and load schema

| Artifact location | Type | Schema to load |
|-------------------|------|----------------|
| `specs/proposed/` | Spec | `~/deploy/dev-workflow/schemas/spec.md` |
| `docs/VISION.md` | Vision | `~/deploy/dev-workflow/schemas/vision.md` |
| `docs/SCOPE.md` | Scope | `~/deploy/dev-workflow/schemas/scope.md` |
| `docs/ROADMAP.md` | Roadmap | `~/deploy/dev-workflow/schemas/roadmap.md` |
| Implementation code | Implementation | Read the spec it implements |
| `bugs/fixing/` | Bug fix | `~/deploy/dev-workflow/schemas/bug-report.md` |

Load the relevant schema:
```
Read ~/deploy/dev-workflow/schemas/<type>.md
```

### 4. Perform review

Follow the reviewer role instructions. For each artifact:

1. **Read the artifact thoroughly**
2. **Check against schema** — are all required sections present and well-formed?
3. **Evaluate quality** — is it clear, complete, unambiguous, implementable?
4. **Check upstream alignment** — does it align with vision/scope/roadmap/spec?
5. **Identify issues** — categorize as blocking (must fix) vs. suggestions (nice to have)

### 5. Produce review document

Write the review to `reviews/<type>/<timestamp>-<name>-<VERDICT>.md` following the review schema:
```
Read ~/deploy/dev-workflow/schemas/review.md
```

Verdict must be one of:
- **APPROVED** — artifact meets quality bar, ready to advance
- **CHANGES_REQUESTED** — specific issues must be addressed, then re-review
- **REJECTED** — fundamental problems, needs significant rework

### 6. Gatekeeper action (if approved)

If verdict is APPROVED, perform the state transition:

| Artifact | Transition | Command |
|----------|------------|---------|
| Spec in proposed/ | proposed -> todo | `git mv specs/proposed/<name>.md specs/todo/<name>.md` |
| Implementation (all tests pass) | doing -> done | `git mv specs/doing/<name>.md specs/done/<name>.md` |
| Bug fix (sentinel test added) | fixing -> fixed | `git mv bugs/fixing/<name>.md bugs/fixed/<name>.md` |

Ask user confirmation before executing the state transition.

### 7. Multi-model review

If you're reviewing your own work (same model wrote and reviews), flag this:
"Note: same model wrote and is reviewing this artifact. For higher confidence, consider a cross-model review using a different terminal."

Read `~/deploy/dev-workflow/model-config.json` for the recommended review model.

Report status: **DONE** with verdict and any state transitions performed.
