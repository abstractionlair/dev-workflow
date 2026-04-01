# Specification: Enhanced Review Workflow

**Version:** 1.0
**Status:** Draft
**Created:** 2026-04-01
**Author:** Scott + Claude (workbench session)

## Overview

Three enhancements to the multi-model review workflow: switching from opencode to codex for GPT reviews, adding a peer-disagreement protocol where reviewers defend contradictory findings, and implementing cross-model review grading to build per-model calibration data over time.

These changes are motivated by review grade analysis showing open models failing at 0-67% pass rates (now dropped), and the need for structured feedback to optimize per-model review prompts.

## 1. Codex as GPT Harness

### What changes

Replace `opencode` with `codex` CLI for all GPT-model reviews. OpenCode was the wrapper for open models (now dropped) and GPT. Codex is OpenAI's native CLI and provides better GPT integration.

### Invocation pattern

Initial review:
```bash
codex exec -m gpt-5.4 \
  --sandbox read-only \
  --full-auto \
  --skip-git-repo-check \
  --json \
  "review prompt" 2>/dev/null
```

- `--json` emits JSONL; first line contains `thread_id` for session resume
- `--sandbox read-only` for reviews (no writes needed)
- `--full-auto` for non-interactive execution
- `2>/dev/null` suppresses thinking tokens on stderr (context window protection)

### Session ID capture

All three CLIs support race-safe session resume via JSON output:

| CLI | Flag | ID field | Resume |
|-----|------|----------|--------|
| codex | `--json` | `thread_id` (first JSONL line) | `codex exec resume <id> "prompt"` |
| claude | `--output-format json` | `session_id` | `claude --resume <id>` |
| gemini | `-o json` | `session_id` | `gemini --resume <id>` |

Session IDs must always be used for resume — never `--last`, `--continue`, or index-based resume. The VPS runs multiple concurrent sessions; `--last` has a race condition.

### model-config.json changes

```json
"macro-reviewer": {
  "tool_by_artifact": {
    "skeleton": "codex",
    "tests": "codex",
    "implementation": "codex"
  },
  "model_by_artifact": {
    "skeleton": "codex/gpt-5.4",
    "tests": "codex/gpt-5.4",
    "implementation": "codex/gpt-5.4"
  }
}
```

The `tools` section adds:
```json
"codex": {
  "cli": "codex",
  "flags": "--sandbox read-only --full-auto --skip-git-repo-check --json",
  "stderr": "suppress",
  "session_id_path": "thread_id"
}
```

### Authentication

All three CLIs use subscription credentials stored locally (not API keys). No vault integration or environment variables needed for the CLI invocations themselves.

## 2. Peer-Disagreement Protocol

### When it triggers

After all independent reviews complete and the synthesizer reads them, but before final synthesis. The synthesizer identifies **inter-reviewer contradictions**: cases where one reviewer says something is fine and another flags it as a problem, or where reviewers assign materially different severities to the same issue.

The synthesizer does NOT challenge findings based on its own opinion. It stays neutral — a referee, not a participant.

### What does not trigger it

- A finding unique to one reviewer that others didn't mention (discovery, not contradiction)
- Reviewers agreeing at different levels of detail (complementary, not contradictory)
- Differences in prose style or organization

### The follow-up

For each identified contradiction, the synthesizer resumes the sessions of the reviewers involved (typically two, but could be more if multiple reviewers took conflicting positions):

```
This is the review synthesizer following up. Reviewer [model] assessed
[file:line / topic] differently from you:

Your finding: [summary]
Their finding: [summary]

Please respond with:
1. Whether you stand by your assessment
2. Specific evidence (file:line, code quotes, documentation) supporting your position
3. Whether the other reviewer's perspective changes your assessment
```

The synthesizer identifies itself as "the review synthesizer," not by model name. It presents the contradiction factually without indicating which side it favors.

### Incorporating responses

The synthesizer weighs the follow-up responses:
- If a reviewer concedes, the contradiction is resolved
- If all parties defend with evidence, the synthesizer records all positions and the evidence in the synthesis, and flags the item for human attention
- If a reviewer can't provide evidence for their position, that weakens their finding

### Session management

The orchestrator captures session IDs from all review runs (step 1) and passes them to the synthesizer. The synthesizer uses explicit session IDs for all follow-ups.

## 3. Cross-Model Review Grading

### Current state

`review_grades` table: model_name, review_type, grade (EXCELLENT/ADEQUATE/INADEQUATE/HARMFUL), note (freeform text), created_at.

Grades are assigned by the synthesizer in the same pass as synthesis. One grade per review.

### Enhanced grading

After synthesis is complete, each reviewer model grades all reviews independently in a separate session. This produces a multi-perspective grade matrix.

#### Schema changes

```sql
ALTER TABLE review_grades
  ADD COLUMN grader_model TEXT,
  ADD COLUMN failure_mode TEXT,
  ADD COLUMN review_target TEXT,
  ADD COLUMN peer_disagreement JSONB;

-- failure_mode enum
ALTER TABLE review_grades
  ADD CONSTRAINT review_grades_failure_mode_check
  CHECK (failure_mode IS NULL OR failure_mode = ANY(ARRAY[
    'no_output', 'false_positive', 'false_negative',
    'wrong_severity', 'hallucinated_evidence', 'credulous', 'shallow'
  ]));
```

- `grader_model`: which model assigned this grade. NULL for legacy rows (synthesizer-assigned). Enables cross-grading analysis.
- `failure_mode`: structured classification of what went wrong. NULL when grade is EXCELLENT or ADEQUATE.
- `review_target`: what was reviewed (spec name, file paths, commit range). Enables correlation between artifact type and review quality.
- `peer_disagreement`: JSON recording any peer follow-up on this review's findings. Structure: `{"contradictions": [{"topic": "...", "other_reviewer": "...", "resolution": "conceded|defended|unresolved"}]}`. NULL if no contradictions.

#### Grading process

1. Synthesis completes, producing merged findings — these are delivered to the review requester immediately
2. Asynchronously (the review requester does not wait): each reviewer model is given all reviews and the merged findings in a fresh session
3. Each grades every review (including its own) on the standard scale, with failure_mode
4. All grades are stored with `grader_model` populated

Grading is decoupled from the review cycle. It feeds the optimization loop, not the current review.

#### Grading prompt structure

```
You are grading code reviews for quality. You have access to:
- The code that was reviewed
- All reviews produced by different models
- The synthesized findings (the "answer key," though it may also have errors)

For each review, assign:
- Grade: EXCELLENT / ADEQUATE / INADEQUATE / HARMFUL
- Failure mode (if INADEQUATE/HARMFUL): [enum values]
- Brief justification (2-3 sentences)

Grade criteria:
- EXCELLENT: found significant issues others missed, or notably thorough
- ADEQUATE: reasonable findings, no major errors
- INADEQUATE: missed important issues, produced false findings, or no output
- HARMFUL: actively misleading — would cause someone to make worse decisions

You are grading the REVIEW, not the code. A review that correctly identifies
a small issue is ADEQUATE even if it misses other things. A review that
confidently declares something correct when it isn't is HARMFUL.
```

#### Cost management

Full cross-grading (N models × N reviews) runs for the first ~20 review cycles to build calibration data. After that, drop to:
- Synthesizer-only grading for routine reviews
- Full cross-grading when the synthesis found contradictions
- Periodic spot-checks (e.g., every 5th review cycle)

### Feedback loop

Per-model failure mode distributions are queried periodically:

```sql
SELECT model_name, failure_mode, COUNT(*)
FROM review_grades
WHERE failure_mode IS NOT NULL
  AND created_at > NOW() - INTERVAL '30 days'
GROUP BY model_name, failure_mode
ORDER BY model_name, COUNT(*) DESC;
```

Results inform updates to per-model guidance files (`guidance/gemini-reviewer.md`, `guidance/gpt-reviewer.md`). If Gemini's top failure mode is `false_positive`, the guidance emphasizes mechanical verification before asserting findings. If GPT's is `shallow`, the guidance emphasizes depth.

## Acceptance Criteria

### Codex switch
1. GPT reviews use `codex exec` instead of `opencode`
2. Session IDs are captured from JSON output for all CLI invocations
3. model-config.json and workflow-review skill updated

### Peer-disagreement protocol
5. Synthesizer identifies inter-reviewer contradictions before final synthesis
6. Contradicted reviewers are resumed by session ID with the opposing finding
7. Follow-up responses are incorporated into the synthesis
8. Synthesizer remains neutral — does not challenge based on its own assessment
9. Contradictions and resolutions are recorded in `peer_disagreement` column

### Cross-model review grading
10. Schema migration adds grader_model, failure_mode, review_target, peer_disagreement columns
11. Each reviewer model can grade all reviews in a separate session
12. Grades with grader_model populated are stored alongside synthesizer grades
13. Failure mode distribution is queryable per model

## Implementation Notes

- The peer-disagreement protocol adds latency to the review cycle (additional round-trips). This is acceptable — review quality matters more than review speed.
- Cross-grading is fully async — runs after synthesis is delivered. The review requester moves on immediately.
- All three CLIs authenticate via locally stored subscription credentials — no API keys or vault integration needed for invocation.
- The peer-disagreement follow-up prompt should not reveal which model produced which review. Present findings neutrally to avoid model-identity bias.

## Out of Scope

- Automated guidance file updates (model-assisted rewriting of gemini-reviewer.md etc.) — future enhancement once enough calibration data exists
- Retrospective grading against production bugs — valuable but requires a different feedback loop
- Changing the set of reviewer models beyond the codex switch — current set (Claude, Gemini, GPT-5.4) is stable
