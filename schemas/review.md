# Schema: Review

A gatekeeping artifact that ensures quality and completeness before artifacts advance through workflow stages.

## Document Header

```markdown
# [Artifact Type] Review: [Artifact Name]

**Artifact:** [Path to artifact being reviewed]
**Version:** [Version of artifact]
**Reviewer:** [Name or AI model identifier]
**Review Date:** [ISO 8601 date]
**Decision:** [APPROVED | NEEDS CHANGES | REJECTED]
```

- **Artifact Type:** Vision, Scope, Roadmap, Spec, Skeleton, Tests, Implementation, Bug Fix
- **Decision values:**
  - **APPROVED** -- meets all quality standards, ready to proceed
  - **NEEDS CHANGES** -- has issues that must be addressed before proceeding
  - **REJECTED** -- fundamentally flawed, requires major rework

**File location:** `reviews/<type>/` (e.g., `reviews/specs/`, `reviews/implementations/`)
**Naming:** `[artifact-name]-[version]-review-[timestamp].md`
**Immutability:** Review records are never moved or deleted once created.

## Required Sections

### 1. Summary

1-3 sentence overview of findings and decision. Quick reference for stakeholders.

### 2. Evaluation

One subsection per criterion. Use status indicators for clarity.

```markdown
### [Criterion Name]
**Status:** PASS | CONCERN | FAIL
**Finding:** [Specific observation]
**Evidence:** [Quote, line reference, or example]
**Impact:** [Why this matters -- required for CONCERN and FAIL]
```

### 3. Decision Rationale

2-4 paragraphs explaining why APPROVED, NEEDS CHANGES, or REJECTED. Summarize critical findings. For APPROVED: highlight strengths. For NEEDS CHANGES: prioritize required fixes. For REJECTED: explain fundamental issues.

### 4. Required Changes (only if NEEDS CHANGES)

```markdown
### Change 1: [Title]
**Location:** [File, section, line number]
**Current:** [What's there or what's missing]
**Required:** [Specific change needed]
**Rationale:** [Why this change is necessary]
**Blocking:** YES | NO
```

Each change must be specific and actionable. Distinguish blocking (must fix) from non-blocking (nice to have).

### 5. Recommendations (optional)

Non-blocking suggestions for future improvement. Ideas for next version.

### 6. State Transition (for gatekeeper reviews)

```markdown
## State Transition

**From:** [Current state/path]
**To:** [New state/path]
**Authorized:** YES | NO
**Date:** [Date of transition]
```

| Review Type | From | To |
|-------------|------|-----|
| Spec Review | specs/proposed/ | specs/todo/ |
| Implementation Review | specs/doing/ | specs/done/ |
| Bug Fix Review | bugs/fixing/ | bugs/fixed/ |

## Evaluation Criteria by Artifact Type

### Vision
1. **Problem Statement Clarity** -- problem clearly defined with specifics
2. **Target Audience Defined** -- primary and secondary users identified
3. **Measurable Success Criteria** -- concrete metrics at 6mo/1yr/3yr
4. **Feasibility** -- goals realistic given constraints
5. **Alignment with Constraints** -- respects technical/resource limitations

### Scope
1. **Boundary Clarity** -- clear in/out scope delineation
2. **Constraint Completeness** -- technical, resource, timeline constraints defined
3. **Non-Functional Requirements** -- performance, security, reliability specified
4. **Feasibility vs Vision** -- scope deliverable within vision constraints

### Roadmap
1. **Phase Sequencing Logic** -- features ordered by dependencies and value
2. **Feature Detail Sufficiency** -- each feature has all 6 required fields
3. **Milestone Mapping** -- phases map to vision milestones
4. **Risk Mitigation** -- high-risk items addressed early

### Spec
1. **Acceptance Criteria Testability** -- all criteria can be verified by tests
2. **Interface Contract Completeness** -- all signatures, parameters, returns, exceptions
3. **Error Handling Coverage** -- 3-7 error scenarios defined
4. **Data Structure Definitions** -- all custom types fully defined
5. **Implementation Constraints** -- behavior specified, not implementation
6. **Scenario Concreteness** -- specific Given-When-Then with concrete values

### Tests
1. **Coverage Completeness** -- all acceptance criteria have tests
2. **Test Independence** -- tests can run in any order
3. **Assertion Quality** -- specific assertions, clear failure messages
4. **Coverage Thresholds** -- >80% line, >70% branch projected
5. **Edge Case Coverage** -- boundary conditions, null/empty tested
6. **Error Path Testing** -- all error conditions have tests

### Implementation
1. **Tests Passing** -- all tests GREEN
2. **Code Quality** -- readable, maintainable, follows conventions
3. **Adherence to GUIDELINES.md** -- follows project patterns
4. **No Test Modifications** -- tests unchanged or changes approved
5. **Coverage Achieved** -- >80% line, >70% branch
6. **SYSTEM_MAP.md Consistency** -- follows architectural patterns

### Bug Fix
1. **Root Cause Documented** -- clear explanation in bug report
2. **Sentinel Test Quality** -- fails on old code, passes on new code, specific to bug
3. **Fix Scope Appropriate** -- minimal changes, addresses root cause
4. **Regression Prevention** -- test added to regression suite

## Anti-Patterns

**Vague rejection:** "The spec has some issues" -- must specify exact issues with locations and required fixes.

**Scope creep review:** Requiring features beyond the artifact's scope -- put feature suggestions in Recommendations, not Required Changes.

**Rubber stamp approval:** "Looks good!" with no evaluation evidence -- must evaluate against all criteria with specific findings.

**Bike-shedding:** Blocking on formatting or style trivia -- put minor style notes in Recommendations; focus Required Changes on quality and completeness.

**Implementation review without running tests:** "Code looks correct" without test evidence -- must show test output and coverage numbers.

**Conflating review and design:** Requiring algorithm changes based on preference -- if design conflicts with scope, REJECT with evidence. If it's preference, put in Recommendations.
