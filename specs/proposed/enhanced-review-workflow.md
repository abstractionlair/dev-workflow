# Specification: Enhanced Review Workflow

**Feature ID:** feature_erw
**Version:** 2.0
**Status:** Draft
**Created:** 2026-04-01
**Author:** Workflow maintainer + Claude (workbench session)

## Overview

Three enhancements to the multi-model review workflow: switching from opencode to codex for GPT reviews, adding a peer-disagreement protocol where reviewers defend contradictory findings, and implementing cross-model review grading to build per-model calibration data over time.

These changes are motivated by review grade analysis showing open models failing at 0-67% pass rates (now dropped), and the need for structured feedback to optimize per-model review prompts.

## Feature Scope

### Included
- Replace opencode with codex CLI for GPT-model reviews
- Race-safe session ID capture and resume across all three CLIs
- Peer-disagreement protocol: synthesizer identifies inter-reviewer contradictions and resumes reviewer sessions for follow-up
- Cross-model review grading: each reviewer model grades all reviews asynchronously after synthesis
- Schema migration for structured failure modes and grading metadata
- Updates to model-config.json, workflow-review skill, bin/workflow-role, and ontology.md

### Excluded (Not in This Feature)
- Automated rewriting of per-model guidance files based on failure mode data
- Retrospective grading against production bugs
- Adding or removing reviewer models beyond the codex switch
- Changes to the micro-reviewer role

### Deferred
- Per-contradiction confidence scoring by the synthesizer
- Automated triggering of full cross-grading based on contradiction count

## User/System Perspective

After this feature, the review orchestrator invokes GPT reviews via `codex exec` instead of `opencode`. When the synthesizer detects contradictions between reviewers, it resumes their sessions to challenge them, producing a richer synthesis. After synthesis is delivered, asynchronous grading sessions produce a multi-perspective quality matrix that accumulates over time, enabling data-driven optimization of per-model review prompts.

## Value Delivered

Replaces a broken open-model review pipeline with a working one (codex/GPT-5.4), adds a mechanism to resolve reviewer disagreements instead of guessing which reviewer is right, and creates a structured feedback loop from review quality back to review prompts. The peer-disagreement protocol directly addresses the most expensive failure mode observed: false negatives where a reviewer confidently declares something correct when it isn't.

## Interface Contract

### Function: extract_session_id(cli: str, raw_output: str) -> str

**Purpose:** Parse session/thread ID from CLI JSON output.

**Parameters:**
- cli (str): Which CLI produced the output. One of "codex", "claude", "gemini".
- raw_output (str): Raw JSON/JSONL output from the CLI invocation.

**Returns:**
- str: The session identifier for resume.

**Raises:**
- ValueError: If raw_output does not contain a parseable session ID.

**Postconditions:**
- Returned ID is safe to pass to the corresponding CLI's resume command.

**Example Usage:**
```python
# Codex: first JSONL line contains thread_id
output = run("codex exec --json ...")
session_id = extract_session_id("codex", output)
# session_id = "019d4686-9f64-77d2-b781-7511612c327c"

# Gemini: JSON object with session_id field
output = run("gemini -o json -p ...")
session_id = extract_session_id("gemini", output)

# Claude: JSON object with session_id field
output = run("claude --output-format json -p ...")
session_id = extract_session_id("claude", output)
```

### Function: detect_contradictions(reviews: list[ReviewOutput]) -> list[Contradiction]

**Purpose:** Identify findings where reviewers reached conflicting assessments of the same code location or topic.

**Parameters:**
- reviews (list[ReviewOutput]): All completed reviews for this review cycle, with findings parsed.

**Returns:**
- list[Contradiction]: Contradictions found. Empty if all reviewers agree.

**Raises:**
- ValueError: If reviews list has fewer than 2 entries.

**Postconditions:**
- Each Contradiction references at least two ReviewOutputs with conflicting positions.
- Findings unique to a single reviewer are NOT included (discovery, not contradiction).
- Findings where reviewers agree at different detail levels are NOT included.

**Example Usage:**
```python
contradictions = detect_contradictions([claude_review, gemini_review, gpt_review])
# Returns e.g. [Contradiction(topic="race condition at db.py:47",
#   positions=[Position(reviewer="claude", ...), Position(reviewer="gemini", ...)])]
```

### Function: request_peer_followup(contradiction: Contradiction, session_ids: dict[str, str]) -> list[FollowupResponse]

**Purpose:** Resume each contradicted reviewer's session and present the opposing finding for response.

**Parameters:**
- contradiction (Contradiction): The contradiction to resolve.
- session_ids (dict[str, str]): Mapping of reviewer name to session ID (e.g. {"claude": "abc-123", "gemini": "def-456"}).

**Returns:**
- list[FollowupResponse]: Each reviewer's response to the challenge.

**Raises:**
- SessionResumeError: If a reviewer's session cannot be resumed (expired, corrupted, etc.).

**Preconditions:**
- All session IDs in session_ids are valid and resumable.

**Postconditions:**
- Each involved reviewer has been presented with the contradiction and asked to respond.
- The follow-up prompt does NOT reveal which model produced which finding (uses "Another reviewer" or "Reviewer A/B").

**Example Usage:**
```python
responses = request_peer_followup(
    contradiction,
    session_ids={"claude": "abc-123", "gemini": "def-456"}
)
# responses[0].reviewer == "claude", responses[0].stance == "conceded"
# responses[1].reviewer == "gemini", responses[1].stance == "defended"
```

### Function: grade_reviews(reviews: list[ReviewOutput], synthesis: SynthesisOutput, grader_cli: str, grader_model: str) -> list[ReviewGrade]

**Purpose:** Have a single model grade all reviews from a review cycle.

**Parameters:**
- reviews (list[ReviewOutput]): All reviews to grade.
- synthesis (SynthesisOutput): The merged findings (serves as approximate answer key).
- grader_cli (str): Which CLI to use for the grading session ("codex", "claude", "gemini").
- grader_model (str): Model identifier for the grading session.

**Returns:**
- list[ReviewGrade]: One grade per review, with structured failure mode.

**Raises:**
- GradingError: If the grader produces unparseable output.

**Postconditions:**
- Each review in the input has exactly one corresponding ReviewGrade in the output.
- Each ReviewGrade has a valid grade enum value and (if INADEQUATE/HARMFUL) a valid failure_mode.

**Example Usage:**
```python
grades = grade_reviews(
    reviews=[claude_review, gemini_review, gpt_review],
    synthesis=merged_findings,
    grader_cli="codex",
    grader_model="gpt-5.4"
)
# grades[0].reviewed_model == "claude", grades[0].grade == "ADEQUATE"
```

## Acceptance Criteria

### Happy Path
1. `codex exec -m gpt-5.4 --json ...` produces JSONL output from which `extract_session_id("codex", ...)` returns a valid thread_id.
2. `gemini -o json -p ...` produces JSON from which `extract_session_id("gemini", ...)` returns a valid session_id.
3. `claude --output-format json -p ...` produces JSON from which `extract_session_id("claude", ...)` returns a valid session_id.
4. Given 3 reviews where reviewer A flags "critical issue at file.py:47" and reviewer B says "file.py:47 is correct," `detect_contradictions` returns one Contradiction referencing both.
5. Given a Contradiction and valid session IDs, `request_peer_followup` resumes both reviewer sessions and returns their responses.
6. The follow-up prompt presented to reviewers uses anonymous labels ("Another reviewer"), not model names.
7. `grade_reviews` returns one ReviewGrade per input review with grade and failure_mode populated.
8. Grades from cross-grading are stored in review_grades with `grader_model` populated.
9. model-config.json references `codex` instead of `opencode` for code artifact reviews.
10. `bin/workflow-role` launches `codex` for macro-reviewer and micro-reviewer code reviews.

### Error Handling
11. `extract_session_id` raises ValueError when CLI output is empty or missing the expected ID field.
12. `extract_session_id` raises ValueError when codex JSONL first line is not valid JSON.
13. `request_peer_followup` raises SessionResumeError when a session ID is expired or invalid, and includes which reviewer failed.
14. `grade_reviews` raises GradingError when the grader model produces output that cannot be parsed into ReviewGrade structures.
15. Schema migration applies cleanly to existing review_grades rows (new columns default to NULL for legacy data).

### Edge Cases
16. `detect_contradictions` returns empty list when all reviewers agree (even at different detail levels).
17. `detect_contradictions` returns empty list when a finding is unique to one reviewer (discovery, not contradiction).
18. A contradiction involving 3 reviewers (A says critical, B says minor, C says not an issue) produces a single Contradiction with 3 positions.
19. Cross-grading handles a reviewer grading its own review — the self-grade is stored with both `model_name` and `grader_model` set to the same value.
20. When a reviewer concedes in peer follow-up, the synthesizer records resolution as "conceded" in the peer_disagreement JSONB.

## Scenarios

### Scenario 1: Clean codex review with session capture

**Given:**
- codex CLI is available with valid subscription credentials
- A spec file exists at specs/proposed/feature-x.md
- model-config.json specifies codex/gpt-5.4 for spec reviews

**When:**
- The orchestrator runs `codex exec -m gpt-5.4 --sandbox read-only --full-auto --skip-git-repo-check --json "Review this spec..." 2>/dev/null`

**Then:**
- JSONL output starts with `{"type":"thread.started","thread_id":"<uuid>"}`
- `extract_session_id("codex", output)` returns the UUID
- The review text is extracted from `item.completed` events with `type: "agent_message"`

### Scenario 2: Peer-disagreement with resolution

**Given:**
- Claude review says "No race condition at db.py:47, the lock is held correctly"
- Gemini review says "Critical: race condition at db.py:47, lock acquired after read"
- GPT review does not mention db.py:47
- All three session IDs are captured

**When:**
- `detect_contradictions` processes all three reviews

**Then:**
- Returns 1 Contradiction with 2 positions (Claude and Gemini)
- GPT is not involved (it didn't assess db.py:47)
- `request_peer_followup` resumes Claude's and Gemini's sessions
- Follow-up prompt to Claude says: "Another reviewer assessed db.py:47 differently from you: they flagged a race condition where the lock is acquired after the read. Your finding: no race condition, lock is held correctly. Please respond with..."
- If Claude concedes: contradiction resolution = "conceded", Gemini's finding stands
- If both defend: synthesizer records both positions and flags for human attention

### Scenario 3: Cross-grading reveals self-rating bias

**Given:**
- A review cycle produced 3 reviews (Claude, Gemini, GPT)
- Synthesis is complete and delivered to the review requester
- Cross-grading cycle count is < 20 (full cross-grading active)

**When:**
- Each reviewer model grades all 3 reviews asynchronously

**Then:**
- 9 ReviewGrade records are created (3 graders × 3 reviews)
- Each has `grader_model` populated
- Gemini grades its own review as ADEQUATE (no failure_mode)
- Claude and GPT both grade Gemini's review as INADEQUATE with failure_mode "false_positive"
- This pattern (self-grade diverging from peer grades) is visible in queries and informs guidance updates

### Scenario 4: Reviewer session expired before follow-up

**Given:**
- A contradiction is detected between Claude and GPT
- Claude's session ID is valid
- GPT's session has expired (codex session TTL exceeded)

**When:**
- `request_peer_followup` attempts to resume both sessions

**Then:**
- Claude's session resumes successfully and returns a response
- GPT's resume raises SessionResumeError with message indicating which reviewer failed
- The synthesizer records the contradiction as "unresolved" for GPT and incorporates only Claude's follow-up

## Data Structures

### ReviewOutput

**Type:** Dataclass

```python
@dataclass
class ReviewOutput:
    reviewer: str           # e.g. "claude", "gemini", "gpt-5.4"
    cli: str                # e.g. "claude", "gemini", "codex"
    session_id: str         # captured from JSON output
    findings: list[Finding] # parsed from review text
    raw_text: str           # full review output
```

**Invariants:**
- reviewer is non-empty
- session_id is a valid UUID or thread ID for the corresponding CLI
- findings may be empty (reviewer produced no findings)

### Finding

**Type:** Dataclass

```python
@dataclass
class Finding:
    topic: str              # what the finding is about
    location: Optional[str] # file:line if applicable
    severity: str           # "critical", "high", "medium", "low"
    assessment: str         # the reviewer's conclusion
```

### Contradiction

**Type:** Dataclass

```python
@dataclass
class Contradiction:
    topic: str                  # what the disagreement is about
    location: Optional[str]     # file:line if applicable
    positions: list[Position]   # at least 2
```

**Invariants:**
- len(positions) >= 2
- All positions reference different reviewers
- Positions represent genuinely conflicting assessments, not different detail levels

### Position

**Type:** Dataclass

```python
@dataclass
class Position:
    reviewer: str       # which reviewer holds this position
    assessment: str     # their finding/conclusion
    severity: str       # their severity rating
```

### FollowupResponse

**Type:** Dataclass

```python
@dataclass
class FollowupResponse:
    reviewer: str           # which reviewer responded
    stance: str             # "conceded", "defended", "unresolved"
    evidence: Optional[str] # file:line, code quotes, or documentation cited
    response_text: str      # full response
```

**Invariants:**
- stance is one of "conceded", "defended", "unresolved"
- "unresolved" is used when the session could not be resumed

### ReviewGrade

**Type:** Dataclass

```python
@dataclass
class ReviewGrade:
    reviewed_model: str             # model whose review is being graded
    grader_model: str               # model doing the grading
    grade: str                      # EXCELLENT, ADEQUATE, INADEQUATE, HARMFUL
    failure_mode: Optional[str]     # null unless INADEQUATE/HARMFUL
    note: str                       # 2-3 sentence justification
    review_target: str              # what was reviewed (spec name, file paths)
    peer_disagreement: Optional[dict]  # contradiction data if applicable
```

**Invariants:**
- grade is one of EXCELLENT, ADEQUATE, INADEQUATE, HARMFUL
- failure_mode is null when grade is EXCELLENT or ADEQUATE
- failure_mode is one of: no_output, false_positive, false_negative, wrong_severity, hallucinated_evidence, credulous, shallow
- note is non-empty

### Database schema changes

```sql
ALTER TABLE review_grades
  ADD COLUMN grader_model TEXT,
  ADD COLUMN failure_mode TEXT,
  ADD COLUMN review_target TEXT,
  ADD COLUMN peer_disagreement JSONB;

ALTER TABLE review_grades
  ADD CONSTRAINT review_grades_failure_mode_check
  CHECK (failure_mode IS NULL OR failure_mode = ANY(ARRAY[
    'no_output', 'false_positive', 'false_negative',
    'wrong_severity', 'hallucinated_evidence', 'credulous', 'shallow'
  ]));
```

New columns default to NULL, so existing rows are unaffected.

## Dependencies

### Internal Dependencies
- `model-config.json` — tool_by_artifact and model_by_artifact entries for code reviews (currently references opencode, must change to codex)
- `bin/workflow-role` — harness selection logic (lines ~100-130 reference opencode, must add codex)
- `roles/macro-reviewer.md` — default-model frontmatter references opencode (must update)
- `ontology.md` — review matrix references opencode (must update)
- `skills/workflow-review/SKILL.md` — references opencode in multi-model note (must update)
- `review_grades` table in claude_hub Postgres — schema migration target

### External Dependencies
- `codex` CLI (npm: @openai/codex) — installed globally
- `claude` CLI — installed locally
- `gemini` CLI (npm: @google/gemini-cli) — installed globally
- All three CLIs authenticated via subscription credentials (no API keys needed)

### Assumptions
- Codex session files persist long enough for peer-disagreement follow-up (typically minutes, not hours)
- All three CLIs continue to support JSON output with session IDs in their current format
- The review_grades table exists and is writable via the claude_hub database

## Constraints and Limitations

### Technical Constraints
- Peer-disagreement follow-ups require the original CLI session to still be resumable. Sessions that expire before follow-up result in "unresolved" contradictions.
- Cross-grading is bounded by token cost: N graders × N reviews per cycle.

### Known Limitations
- The synthesizer cannot detect contradictions that are semantically equivalent but expressed in different terms (e.g., "lock ordering issue" vs "race condition at the same location"). Contradiction detection relies on topic/location overlap.
- Cross-grading calibration data is only as good as the graders. If all three models share a blind spot, cross-grading won't reveal it.

### Out of Scope
- Automated guidance file updates from failure mode data
- Retrospective grading against production bugs
- Changes to the set of reviewer models beyond the codex switch

## Implementation Notes

- The peer-disagreement protocol adds latency to the review cycle (additional round-trips). This is acceptable — review quality matters more than review speed.
- Cross-grading is fully async — runs after synthesis is delivered. The review requester moves on immediately.
- All three CLIs authenticate via locally stored subscription credentials — no API keys or vault integration needed for invocation.
- The follow-up prompt must use anonymous labels ("Another reviewer", "Reviewer A"), never model names, to avoid model-identity bias.
- `2>/dev/null` on codex invocations suppresses thinking tokens emitted on stderr.

### Cross-grading policy

Full cross-grading (every grader grades every review) runs for the first 20 completed review cycles, tracked by a counter in the review_grades table:

```sql
SELECT COUNT(DISTINCT job_id) FROM review_grades WHERE grader_model IS NOT NULL;
```

After 20 cycles:
- Synthesizer-only grading for routine reviews (grader_model = synthesizer model)
- Full cross-grading when the synthesis found at least one contradiction
- Full cross-grading on every 5th cycle (cycle count mod 5 = 0)

## Open Questions

- [ ] Should the follow-up prompt include the original code context, or rely on the reviewer's session having it in context already?
- [ ] What is the session TTL for each CLI? This determines the window for peer-disagreement follow-ups.
- [ ] Should cross-grading results be surfaced to the review requester, or kept as background calibration data only?

## References

- This project (dev-workflow) skips strategic planning stages — no VISION.md, SCOPE.md, or ROADMAP.md exist. This spec is a standalone feature enhancement.
- Review grade analysis: `SELECT model_name, grade, COUNT(*) FROM review_grades GROUP BY model_name, grade` — data that motivated these changes.
- Prior art: [skill-codex](https://github.com/skills-directory/skill-codex) — peer-disagreement protocol inspiration.
