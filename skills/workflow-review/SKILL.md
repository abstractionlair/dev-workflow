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

When the user requests review by multiple models (e.g., "review by all frontier models", "multi-model review"), this session acts as **orchestrator**. The orchestrator launches reviews in parallel, detects contradictions, drives peer follow-up, synthesizes, and triggers cross-grading.

### 8. Launch reviews with session capture

For each reviewer model, launch via `workflow-role` and capture the session ID for potential follow-up:

```bash
# Claude (this session does its own review — steps 1-7 above)

# Codex/GPT — capture session ID from JSONL output
codex exec -m gpt-5.4 --sandbox read-only --skip-git-repo-check --json \
  "$(cat .workflow/active-role.md)" 2>/dev/null | tee /tmp/review-codex.jsonl
CODEX_SESSION=$(extract-session-id codex < /tmp/review-codex.jsonl)

# Gemini — capture session ID from JSON output
gemini -o json -p "$(cat .workflow/active-role.md)" | tee /tmp/review-gemini.json
GEMINI_SESSION=$(extract-session-id gemini < /tmp/review-gemini.json)
```

Alternatively, launch via `workflow-role` interactively and note the session ID from the output.

Store session IDs — they're needed for peer follow-up.

### 9. Collect reviews

Read all reviews from `reviews/<category>/` for this artifact. Each review should be from a different model.

### 10. Detect contradictions

Compare findings across reviews. A **contradiction** is where two or more reviewers reached conflicting assessments of the same code location or topic:

- Reviewer A says "no race condition at db.py:47" while Reviewer B says "critical race condition at db.py:47" → **contradiction**
- Reviewer A flags something that no other reviewer mentions → **not a contradiction** (unique discovery)
- Reviewer A says "minor issue at line 50" while Reviewer B says "critical issue at line 50" about the same thing → **contradiction** (conflicting severity)
- Reviewers agree on an issue but describe it at different detail levels → **not a contradiction**

List each contradiction with: topic, location (if applicable), and each reviewer's position.

### 11. Peer-disagreement follow-up

For each contradiction, resume the involved reviewers' sessions and present the opposing finding. The synthesizer (this session) acts as a **neutral referee** — it does not take sides.

**Follow-up prompt template** (sent to each contradicted reviewer via session resume):

> Another reviewer assessed `[location/topic]` differently from you.
>
> Their finding: [opposing assessment]
>
> Your finding: [this reviewer's assessment]
>
> Please respond:
> 1. Do you **concede** (they're right, you were wrong)?
> 2. Do you **defend** your position (cite specific evidence — code lines, docs, logic)?
> 3. If defending, what specific evidence supports your position?

**Critical:** Use anonymous labels ("Another reviewer"), never model names. This prevents model-identity bias.

**Resume commands:**
```bash
# Resume codex session
codex exec --thread "$CODEX_SESSION" --json "Follow-up prompt..." 2>/dev/null

# Resume claude session
claude --session-id "$CLAUDE_SESSION" --output-format json -p "Follow-up prompt..."

# Resume gemini session
gemini --session "$GEMINI_SESSION" -o json -p "Follow-up prompt..."
```

Record each response as: **conceded**, **defended**, or **unresolved** (session expired/failed).

### 12. Synthesize

Merge all reviews and follow-up responses into a synthesis:

1. Findings where all reviewers agree → include directly
2. Unique discoveries (one reviewer only) → include, note single-source
3. Resolved contradictions (one side conceded) → include the winning position
4. Unresolved contradictions (both defend or session failed) → flag for human attention with both positions

Write the synthesis to `reviews/<category>/` with the usual naming convention.

### 13. Cross-grading (async)

After delivering the synthesis to the review requester, launch cross-grading **asynchronously**. The requester does not wait for grading results.

**When to do full cross-grading** (every grader grades every review):
- First 20 completed review cycles (check: `SELECT COUNT(DISTINCT job_id) FROM review_grades WHERE grader_model IS NOT NULL`)
- Any cycle where contradictions were found
- Every 5th cycle thereafter

**Otherwise:** Synthesizer-only grading (this session grades all reviews).

**Grading prompt** (sent to each grader model in a fresh session):

> You are grading the quality of code reviews. For each review below, assign:
>
> **Grade:** EXCELLENT | ADEQUATE | INADEQUATE | HARMFUL
> **Failure mode** (if INADEQUATE/HARMFUL): no_output | false_positive | false_negative | wrong_severity | hallucinated_evidence | credulous | shallow
> **Note:** 2-3 sentence justification
>
> The synthesis below represents the merged consensus. Use it as an approximate answer key.
>
> [Include: each review text, the synthesis, and the artifact under review]

**Store grades** in the `review_grades` table. Note: this table lives in external infrastructure (a Postgres instance maintained outside this repo) — the repo ships neither its schema nor migrations, so without an equivalent database of your own, skip grade storage:

```sql
INSERT INTO review_grades (job_id, model_name, review_type, grade, note, grader_model, failure_mode, review_target, peer_disagreement)
VALUES ($job_id, $reviewed_model, $review_type, $grade, $note, $grader_model, $failure_mode, $artifact_path, $peer_data);
```

- `model_name`: the model whose review is being graded
- `grader_model`: the model doing the grading
- `peer_disagreement`: JSONB with contradiction data if applicable (topic, positions, resolution)
- All grades from one grading session share the same `job_id`
