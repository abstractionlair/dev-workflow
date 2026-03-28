# Schema: Spec

A behavioral contract defining what to build, consumed by Skeleton Writer, Test Writer, and Implementer.

## Document Header

```markdown
# Specification: [Feature Name]

**Feature ID:** [Unique identifier from roadmap]
**Version:** [Semantic version]
**Status:** [Draft | Review | Approved | Implemented]
**Created:** [Date]
**Author:** [Name or role]
```

- **Feature Name:** Matches ROADMAP.md entry, 2-5 word noun phrase
- **Feature ID:** Format `feature_NNN` or custom scheme, used for traceability
- **Version:** Semantic versioning (1.0, 1.1, 2.0), increment on changes
- **Status:** Draft (WIP) | Review (ready for review) | Approved (ready to implement) | Implemented (tests passing)

**File location:** `specs/proposed/` (moves through `todo/` -> `doing/` -> `done/`)

## Required Sections (in order)

### 1. Overview

1-2 paragraphs: what this feature enables and why it matters. Derives from ROADMAP.md description.

```markdown
## Overview

The Context Linking Engine automatically discovers and maintains relationships
between specifications, tests, and implementation code. It analyzes the codebase
to detect references based on naming conventions, decorators, and comments.

This eliminates the "where does this spec live in code?" problem, reducing
search time from 5-30 minutes to under 10 seconds.
```

### 2. Feature Scope

```markdown
## Feature Scope

### Included
- [Core capability 1]
- [Core capability 2]

### Excluded (Not in This Feature)
- [Related feature that's out of scope]

### Deferred (Maybe in This Feature)
- [Optional enhancement if time allows]
```

- **Included:** 3-7 specific, observable behaviors
- **Excluded:** Prevents scope creep, clarifies boundaries
- **Deferred:** Only if time allows after core complete

### 3. User/System Perspective

Observable changes from user or system viewpoint. Focus on what users can do after feature exists. No implementation details.

### 4. Value Delivered

Problem solved and improvement provided. Quantify impact if possible. Reference VISION.md if applicable.

### 5. Interface Contract

**Critical: This is what Skeleton Writer and Test Writer consume.**

#### For Functions:

```markdown
### Function: function_name(params) -> ReturnType

**Purpose:** [One sentence]

**Parameters:**
- param_name (Type): Description
  - Constraints: [Valid values, ranges]
  - Example: [Concrete value]
  - Default: [If applicable]

**Returns:**
- Type: Description
  - Structure: [Format or shape]
  - Example: [Concrete value]

**Raises:**
- ExceptionType: [Trigger condition]

**Preconditions:**
- [What must be true before calling]

**Postconditions:**
- [What is guaranteed after success]

**Example Usage:**
```python
result = function_name(param="value")
```
```

#### For Classes:

```markdown
### Class: ClassName

**Purpose:** [What this class represents]

**Attributes:**
- attribute_name (Type): Description
  - Invariants: [Rules that must hold]

**Methods:**

#### __init__(self, param: Type) -> None
[Constructor spec in function format]

#### method_name(self, param: Type) -> ReturnType
[Method spec in function format]

**Invariants:**
- [Class-level invariant]
```

**Requirements:**
- Every public function/method specified
- Every parameter has type and description
- Every return has type and description
- Every exception documented with trigger condition
- Concrete examples for complex interfaces

### 6. Acceptance Criteria

**Critical: This becomes the test suite.**

```markdown
## Acceptance Criteria

### Happy Path
1. [Observable criterion verifiable by test]
2. [Another criterion for normal operation]

### Error Handling
3. [Criterion for specific error case]
4. [Criterion for validation failure]

### Edge Cases
5. [Criterion for boundary condition]
6. [Criterion for empty/null input]

### Performance (if applicable)
7. [Response time requirement]

### State Management (if applicable)
8. [Criterion for state persistence]
```

Each criterion must be:
1. **Specific:** No ambiguity about what to verify
2. **Testable:** Can write automated test that checks it
3. **Observable:** Result can be measured or inspected
4. **Independent:** Can be verified in isolation

Good vs bad:
```
GOOD: "Scanning 10 specs and 20 code files completes in <5 seconds"
GOOD: "Raises InvalidPathError with message 'Project path does not exist: [path]' for missing directories"
BAD:  "System handles input appropriately"
BAD:  "Errors are clear"
```

**Coverage targets:** 2-5 happy path, 3-7 error handling, 3-7 edge cases. 10-20 total.

### 7. Scenarios

Given-When-Then format with concrete values.

```markdown
### Scenario 1: Basic Link Detection

**Given:**
- Project with /specs/feature_001_user_login.md
- Code file /src/auth/login.py containing "# Implements: feature_001"
- Test file /tests/test_login.py with @spec("feature_001") decorator

**When:**
- LinkingEngine.scan() is called

**Then:**
- LinkMap contains: "feature_001" -> {"code": ["src/auth/login.py"], "tests": ["tests/test_login.py"]}
- Scan completes in <5 seconds
```

**Requirements:**
- 3-7 scenarios minimum
- At least 1 happy path, 2 error, 1 edge case
- Use concrete values, not variables
- Each scenario maps to 1-3 acceptance criteria

### 8. Data Structures

```markdown
### LinkMap

**Type:** Dictionary[str, LinkEntry]

**Structure:**
```python
{
    "spec_id_1": {"code": ["file1.py"], "tests": ["test1.py"]},
    "spec_id_2": {"code": [], "tests": ["test2.py"]}
}
```

**Invariants:**
- All spec IDs are non-empty strings
- Code and test lists may be empty but never null
- No duplicate paths within lists
```

Every custom type must be defined with structure, invariants, and a concrete example.

### 9. Dependencies

```markdown
## Dependencies

### External Dependencies
- [Library/package and version]

### Internal Dependencies
- [Other feature/module this depends on]

### Assumptions
- [What we're assuming about the environment]
```

### 10. Constraints and Limitations

```markdown
### Technical Constraints
- [Hard limits from design or technology]

### Language/Platform Constraints
- [Platform-specific limitations]

### Known Limitations
- [What this feature doesn't handle]

### Out of Scope
- [Explicitly excluded capabilities]
```

### 11. Implementation Notes (Optional)

Guidance, not requirements. Suggested approaches, performance tips, error handling patterns. Implementation is free to differ.

### 12. Open Questions

```markdown
- [ ] [Question needing clarification before implementation]
- [ ] [Decision point requiring stakeholder input]
```

Resolve and remove as answers are found.

### 13. References

Required links: ROADMAP.md (source feature), VISION.md (strategic context), SCOPE.md (boundary context). Optional: related specs, SYSTEM_MAP.md, GUIDELINES.md.

## Downstream Consumption

| Consumer | Reads | Creates |
|----------|-------|---------|
| Skeleton Writer | Interface Contract, Data Structures, Exceptions | Interface files, type definitions, exception classes |
| Test Writer | Acceptance Criteria, Scenarios, Interface Contract | Test suite (one test per criterion), scenario tests |
| Implementer | Interface Contract, Acceptance Criteria, Constraints | Working code that makes tests pass |

## Anti-Patterns

**Vague acceptance criteria:** "System handles input appropriately" -- must specify exact inputs, outputs, and error conditions.

**Missing error conditions:** Only happy path specified -- add Error Handling section with 3-7 error criteria.

**Implementation in interface contract:** Specifies algorithms or data structures -- describe behavior (inputs/outputs/errors) not implementation.

**Coupled acceptance criteria:** "System creates user AND sends email AND logs event" -- split into independent criteria.

**Missing data structure definitions:** Uses custom types without defining them -- Skeleton Writer cannot create types without definitions.

## Behavioral Focus

- **MUST:** Describe observable behavior, focus on WHAT not HOW, use domain language, include error conditions
- **MUST NOT:** Prescribe implementation, include code (except Interface Contract examples), be vague, omit error handling
