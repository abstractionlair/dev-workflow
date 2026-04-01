# Spec Review: Enhanced Review Workflow

**Reviewer**: Gemini
**Date**: 2026-04-01
**Artifact**: specs/proposed/enhanced-review-workflow.md
**Status**: CHANGES_REQUESTED

## Summary

The design for the Enhanced Review Workflow is well-motivated and technically sound, addressing clear pain points identified in previous review cycles (e.g., model failure rates and the need for better calibration). The peer-disagreement protocol and cross-model grading are significant improvements to the multi-model review strategy. However, the specification fails to meet the structural and behavioral requirements of the `Spec` schema. It is currently written more as a design proposal than a behavioral contract for downstream consumers. Most critically, it lacks the formal interface contracts and Given-When-Then scenarios required for the Skeleton Writer and Test Writer to proceed without clarification.

## Checklist

- [x] Document Header: Feature Name, Version, Status, Created, Author (Missing Feature ID)
- [x] Overview: Present and well-motivated
- [ ] Feature Scope: Present but missing required "Included/Excluded/Deferred" structure
- [ ] User/System Perspective: **Missing**
- [ ] Value Delivered: **Missing** (partially covered in Overview)
- [ ] Interface Contract: **Missing** (CRITICAL: no function/method signatures)
- [ ] Acceptance Criteria: Present but not grouped by required categories (Happy Path, Error, Edge)
- [ ] Scenarios: **Missing** (CRITICAL)
- [ ] Data Structures: Present but not in the required schema format (Type/Structure/Invariants)
- [ ] Dependencies: Mentioned in notes but missing dedicated section
- [ ] Constraints and Limitations: **Missing** (Out of Scope is present)
- [x] Implementation Notes: Present and useful
- [ ] Open Questions: **Missing**
- [ ] References: **Missing** (No links to ROADMAP.md, VISION.md, SCOPE.md)

## Critical Issues (P0 -- blocks progress)

1. **Missing Interface Contract:** The spec describes processes narratively but does not define formal function or method signatures for the peer-disagreement orchestrator or the cross-grading engine. A Skeleton Writer cannot derive types, parameters, or return values from the current text. 
   - **Fix:** Add an `Interface Contract` section defining specific functions (e.g., `Synthesizer.identify_contradictions()`, `Synthesizer.request_followup(session_id, contradiction)`) with full type signatures, parameter descriptions, and return types.

2. **Missing Scenarios:** There are no Given-When-Then scenarios. These are essential for the Test Writer to understand the expected behavior of the peer-disagreement protocol, particularly what triggers a contradiction versus a discovery.
   - **Fix:** Add a `Scenarios` section with 3-7 scenarios covering happy paths (clear contradiction), edge cases (agreement at different detail levels), and error cases.

3. **Missing Feature ID and Upstream Alignment:** The spec does not provide a `Feature ID` and does not link to the required upstream artifacts (`ROADMAP.md`, `VISION.md`, `SCOPE.md`). This breaks traceability.
   - **Fix:** Assign a Feature ID (e.g., `feature_042`) and add a `References` section with links to the upstream planning documents.

## Improvements (P1 -- reduces effectiveness)

1. **Structural Alignment:** Several sections are either missing or not using the required naming and structure from the `schemas/spec.md`.
   - **Fix:** Reorganize the document to include all required sections in order, specifically using the "Included/Excluded/Deferred" format for `Feature Scope`.

2. **Data Structure Definitions:** The database schema changes and JSON structures are provided as raw code/SQL, which is helpful, but they lack the required definitions for invariants and concrete examples in the spec format.
   - **Fix:** Use the `Data Structures` format from the schema for the `peer_disagreement` JSONB column and any internal objects.

3. **Contradictory Follow-up Template:** The prompt template for peer-disagreement follow-ups includes "Reviewer [model]", which directly contradicts the implementation note that says "should not reveal which model produced which review."
   - **Fix:** Update the template to use neutral labels like "Reviewer A" or "The other reviewer" to maintain the "neutral referee" principle.

4. **Missing Acceptance Criteria Coverage:** The criteria are not grouped by Happy Path, Error Handling, and Edge Cases, making it difficult to verify full coverage. Criterion 4 is also missing from the numbering.
   - **Fix:** Regroup the criteria and ensure at least 3-7 criteria each for Error Handling and Edge Cases.

## Strengths

- **Empirically Motivated:** The switch to Codex and the grading enhancements are directly supported by the mention of `review_grades` analysis.
- **Cost-Conscious:** The pragmatic cost management strategy for cross-grading (tapering off after calibration) is excellent.
- **Precise Tooling:** The "Session ID capture" table is a high-signal addition that provides clear guidance for the orchestrator implementation.

## Decision

CHANGES_REQUESTED: The spec is a strong design document but a weak behavioral contract. It requires formal interface definitions, scenarios, and structural alignment with the spec schema before it can be handed off to skeleton and test writers.
