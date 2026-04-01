# Spec Review: Enhanced Review Workflow

**Reviewer**: Codex (GPT-5)
**Date**: 2026-04-01
**Artifact**: specs/proposed/enhanced-review-workflow.md
**Status**: CHANGES_REQUESTED

## Summary
The design direction is strong. The codex switch is motivated by current repository state, the session-resume guidance is concrete, and the peer-disagreement idea is a meaningful quality improvement rather than process theater. The cross-grading loop is also strategically aligned with the stated goal of calibrating reviewers over time.

The problem is that this document is still a design note, not a spec contract. Against `schemas/spec.md`, it is missing multiple mandatory sections and does not define the interfaces, data structures, scenarios, error cases, or upstream traceability that Skeleton Writer, Test Writer, and Implementer need. Confidence is therefore low: an implementer could build something plausible from this, but not something reliably consistent with the intended behavior.

## Checklist
- [ ] Header complete per schema (`Feature ID` missing)
- [ ] Required sections present in schema order
- [ ] Feature Scope explicitly split into Included / Excluded / Deferred
- [ ] User/System Perspective present
- [ ] Value Delivered present
- [ ] Interface Contract present with signatures, types, returns, raises, preconditions, postconditions, examples
- [ ] Acceptance criteria complete across happy path, error handling, and edge cases
- [ ] Scenarios present in Given-When-Then format with concrete values
- [ ] Data structures fully defined with invariants and examples
- [ ] Dependencies identified with concrete internal file paths/functions
- [ ] Constraints and Limitations present
- [ ] Open Questions present
- [ ] References to ROADMAP.md, VISION.md, and SCOPE.md present
- [x] Motivation and intent are clear
- [x] Core workflow ideas appear feasible in principle

## Critical Issues (P0 -- blocks progress)
1. **Schema non-compliance is substantial**: The spec is missing required header and sections: `Feature ID`, `Feature Scope`, `User/System Perspective`, `Value Delivered`, `Interface Contract`, `Scenarios`, `Dependencies`, `Constraints and Limitations`, `Open Questions`, and `References`. `Out of Scope` exists, but not inside the required `Constraints and Limitations` structure. This alone blocks downstream consumers. -> Specific fix: rewrite the document into the exact schema structure from `schemas/spec.md`, preserving the current design content inside the required sections.

2. **No interface contract for the workflow behaviors**: Lines 20-122 and 159-190 describe operations narratively, but there are no public contracts for the actual units an implementer must build: review invocation, session capture, contradiction detection, reviewer follow-up, grade generation, and grade persistence. Without function or method signatures, typed inputs/outputs, and documented exceptions, Skeleton Writer and Test Writer cannot derive artifacts from this spec. -> Specific fix: add explicit contracts for each public workflow entry point, for example `run_review(...)`, `extract_session_id(...)`, `detect_contradictions(...)`, `request_peer_followup(...)`, and `grade_reviews(...)`, each with parameters, return types, failure conditions, and example usage.

3. **Behavior coverage is incomplete and not testable enough**: Acceptance criteria at lines 214-232 cover only broad success statements. They omit required error-handling and edge-case criteria for important failures such as missing `thread_id`/`session_id`, malformed JSON output, reviewer resume failure, ambiguous contradiction matching, duplicate contradictions, grader timeout, and migration behavior on legacy rows. The required Scenarios section is also absent. -> Specific fix: expand acceptance criteria into happy path, error handling, edge cases, and state-management groups with concrete observable outcomes, then add 3-7 Given/When/Then scenarios using actual values.

4. **Upstream alignment cannot be verified, and one core behavior is internally contradictory**: The spec has no References section, no `Feature ID`, and I could not find concrete `ROADMAP.md`, `VISION.md`, or `SCOPE.md` artifacts in the repository to trace this feature against. In addition, lines 99-109 explicitly reveal `Reviewer [model]`, while line 239 says the follow-up prompt must not reveal which model produced which review. -> Specific fix: add traceable references to the upstream artifacts and feature entry, or explicitly block implementation until those artifacts exist; also anonymize the follow-up prompt so it matches the non-disclosure requirement.

## Improvements (P1 -- reduces effectiveness)
1. **Dependencies are under-specified**: The spec names `model-config.json` and `skills/workflow-review/SKILL.md`, but current repository behavior also depends on `bin/workflow-role`, `roles/macro-reviewer.md`, and `ontology.md`, all of which still reference `opencode`. -> Suggested enhancement: add an explicit Dependencies section listing impacted internal files and the behavior each controls.

2. **Data structures are only partially defined**: The DB migration is useful, but the spec does not define the full shapes of contradiction records, follow-up requests/responses, grade payloads, or session-capture records beyond brief prose. -> Suggested enhancement: add typed structures with invariants and concrete examples for `PeerDisagreement`, `Contradiction`, `ReviewGradeInput`, and `ReviewGradeRecord`.

3. **Cross-grading policy needs deterministic rules**: "first ~20 review cycles" and "every 5th review cycle" are directionally fine but not precise enough for implementation and tests. -> Suggested enhancement: specify exactly how cycles are counted, where that state lives, and what happens when asynchronous grading is skipped or retried.

4. **Acceptance criteria do not fully cover repository-visible outcomes**: The codex switch criteria do not mention launcher behavior, even though `bin/workflow-role` currently hard-codes `opencode` selection for code reviews. -> Suggested enhancement: add criteria for end-to-end launch behavior, not just config edits.

## Strengths
The spec does several things well: it is motivated by observed review-grade failures rather than preference, it captures the concurrency/race concern around session resume clearly, and it draws a good boundary around the synthesizer as a neutral referee. The async grading design is also pragmatic because it preserves review latency for the requester while still creating calibration data.

## Decision
CHANGES_REQUESTED -- address the missing schema sections, add real interface contracts and scenarios, and resolve the upstream-traceability and model-identity contradictions before re-review.
