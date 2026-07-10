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
workflow-role macro-reviewer <project> <artifact>   # gemini for docs, codex for code
workflow-role micro-reviewer <project> <artifact>   # codex/gpt-5.4
```

Report status: **DONE** with review type, verdict, and any state transitions.

---

## Multi-Model Orchestrated Review

When the user requests review by multiple models ("review by all frontier models",
"multi-model review"), do NOT orchestrate reviewers by hand. The claude-hub review
engine owns dispatch, synthesis, contradiction detection, peer follow-up, and
reviewer grading. The manual orchestration this section used to describe (launch
sessions, detect contradictions, resume sessions, cross-grade, INSERT grades) is
retired — it was a drift-prone re-implementation of the engine (2026-07-10
consolidation; work-graph ctrs-0).

### 8. Dispatch via the engine

Run from a shell with CLAUDE_HUB_PG_DSN set (the engine stores jobs in Postgres):

```bash
python3 -m claude_hub.review_cli \
  --files <changed files...> \
  --intent-ref <spec or requirements path> \
  --prompt "<review instructions — use the macro- or micro-reviewer role focus>" \
  --review-type <spec|implementation|docs|general> \
  --output reviews/<category>/<artifact>-multimodel.md
```

Notes:
- The model roster lives in claude-hub's `config/review_models.yaml` and is
  authoritative. Do not choose models here; use `--models` only to subset for cost.
- The engine automatically: synthesizes a consensus report, extracts
  contradictions, resumes contradicted reviewers' sessions anonymously
  (concede/defend), grades every reviewer against the failure-mode taxonomy,
  and stores everything (`reviews`, `review_syntheses`, `review_grades`, artifacts).
- NEVER `INSERT INTO review_grades` manually — the engine is the only writer.
- The engine and its Postgres tables are external infrastructure (claude-hub) —
  this repo ships neither the engine nor the schema. Without a claude-hub
  deployment, use single-model review (steps 1-7), varying harness/model for
  cross-model coverage.
- The call blocks until synthesis completes and writes the report to `--output`.

### 9. Deliver

Read the engine's report, write the review document in `reviews/<type>/` per the
schema (the synthesis is your source; attribute single-model findings), and
perform the gatekeeper action (macro review only) as in steps 5-6.
