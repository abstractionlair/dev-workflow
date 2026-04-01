# Spec Review: Enhanced Review Workflow

**Reviewer**: Claude (Opus 4.6) — ⚠️ co-author conflict of interest
**Date**: 2026-04-01
**Artifact**: specs/proposed/enhanced-review-workflow.md
**Status**: CHANGES_REQUESTED

## Summary

The spec clearly captures the design intent from the conversation and the three enhancements are well-motivated by real data (review_grades analysis). The peer-disagreement protocol and cross-model grading are genuinely novel additions to the workflow. However, the spec is shaped more as a design document than a behavioral contract — it lacks the interface contracts, data structures, and scenarios that downstream consumers (skeleton writer, test writer, implementer) need.

Given my co-authorship, I'm biased toward leniency. The Gemini and GPT reviews should be weighted more heavily.

## Checklist

- [x] Overview present and motivated
- [x] Feature scope (implicit, via 3 numbered sections)
- [ ] Interface contract — **missing**
- [ ] Acceptance criteria — present but insufficient (no scenarios, no concrete values)
- [x] Data structures (schema changes defined)
- [x] Dependencies identified (CLI tools, review_grades table)
- [ ] Scenarios (Given/When/Then) — **missing**
- [x] Implementation notes present
- [x] Out of scope defined

## Critical Issues (P0)

1. **No interface contract for the peer-disagreement protocol.** The spec describes the flow narratively but doesn't define what the orchestrator/synthesizer interface looks like. What function is called? What parameters does it take? What does it return? A skeleton writer can't derive interfaces from this.

2. **No interface contract for the cross-grading process.** Same issue — step 2 says "each reviewer model is given all reviews" but doesn't specify the function signature, input format, or output format. The grading prompt is provided (good) but there's no contract for how grades flow back into the database.

3. **Acceptance criteria lack concrete values.** Criterion 5 says "Synthesizer identifies inter-reviewer contradictions" but doesn't define what constitutes a contradiction with testable specificity. What if one reviewer says "minor issue at line 47" and another says "critical issue at line 47"? Is that a contradiction (different severity) or agreement (both found it)?

## Improvements (P1)

1. **Missing scenarios.** The spec schema requires 3-7 Given/When/Then scenarios. These would be especially valuable here because the peer-disagreement protocol has subtle trigger conditions that need concrete examples.

2. **The follow-up prompt template reveals model identity.** The template says "Reviewer [model] assessed..." — but the implementation notes say "should not reveal which model produced which review." These contradict. The template should use anonymous labels ("Another reviewer" or "Reviewer A").

3. **No definition of what "async" means operationally.** The grading is described as async, but there's no specification of *how* it's triggered. Cron? A background process? The orchestrator fires-and-forgets? This matters for implementation.

4. **Acceptance criterion numbering skips 4.** Minor, but suggests editing artifacts.

## Strengths

- The motivation is data-driven — review_grades analysis directly informed the design.
- Session ID capture table is precise and immediately implementable.
- The synthesizer-as-neutral-referee principle is clearly articulated with good boundary definitions (what triggers and what doesn't).
- The cost management strategy for cross-grading is pragmatic.
- The feedback loop from failure_mode distributions to per-model guidance is well-conceived.

## Decision

CHANGES_REQUESTED — The design is sound but the spec needs interface contracts, scenarios, and resolution of the model-identity contradiction before it's ready for a skeleton writer. Given the co-author conflict, defer to the independent reviews for the final verdict.
