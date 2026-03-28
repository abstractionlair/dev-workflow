# Schema: Roadmap

The phased feature sequence bridging strategic planning (Vision, Scope) and tactical execution (Specs).

## Document Header

```markdown
# [Project Name] Roadmap
```

**Produced by:** Planning Writer (after SCOPE.md is complete)
**Consumed by:** Spec Writer (reads feature entries), Reviewer (validates sequencing)
**Lifecycle:** Living document, updated after validation checkpoints. Versioned in place.

## Required Sections (in order)

### 1. Roadmap Overview

2-3 sentences explaining sequencing strategy.

```
This roadmap sequences features to derisk technical approach early while
delivering immediate user value through progressive capability addition.
```

### 2. Alignment

- **Vision:** Copy vision statement verbatim from VISION.md
- **Success Criteria:** Key metrics from VISION.md
- **Scope Summary:** 1-3 sentence overview of in-scope features from SCOPE.md

### 3. Sequencing Strategy

**Key Principles** (3-5):
```markdown
1. Derisk technical linking first - Most novel/risky capability
2. Deliver complete user journeys - Usable value at each phase
3. Validate with real usage - Get features in front of users ASAP
4. Build infrastructure as needed - JIT, not upfront
```

**Risk Mitigation Approach:** Which risks, when mitigated, how validated.

**Value Delivery Pattern:**
```markdown
Each phase delivers a complete user capability:
- Phase 1: Can link code to specs (core value)
- Phase 2: Can create and manage specs
- Phase 3: Can maintain living docs automatically
```

### 4. Phase Definitions (repeat per phase)

```markdown
## Phase [N]: [Phase Name] (Weeks X-Y)
```

Each phase contains:

#### Goals
1-2 paragraphs: outcomes, not features. Value delivered, risks mitigated, assumptions validated.

#### Features

**Critical: This is what Spec Writer consumes.** Each entry uses this exact structure:

```markdown
1. **[Feature Name]**
   - **Description:** [1-2 sentence overview of what this feature does]
   - **Why now:** [Rationale for sequencing in this phase]
   - **Delivers:** [User value or learning outcome]
   - **Derisks:** [Assumptions validated, or "None"]
   - **Depends on:** [Prerequisites, or "Nothing"]
   - **Effort:** [Small (1-3d) | Medium (4-7d) | Large (1.5-4wk)]
```

All 6 fields are required. Feature names are unique within the project, 2-5 word noun phrases.

**Example phase:**
```markdown
## Phase 1: MVP Foundation (Weeks 1-4)

### Goals
Validate core technical approach and deliver first usable capability.

### Features

1. **Static Analysis Engine**
   - **Description:** Analyzes Python code to automatically detect relationships between specs, tests, and implementation files
   - **Why now:** Highest technical risk, must validate before building more
   - **Delivers:** Automatic detection of spec-to-code links
   - **Derisks:** "Static analysis sufficient" assumption
   - **Depends on:** Nothing
   - **Effort:** Large (2 weeks)

2. **Spec Management Commands**
   - **Description:** CLI commands to create, list, and validate specification files
   - **Why now:** Need specs to link to
   - **Delivers:** Can create and manage specs
   - **Derisks:** Template format acceptance
   - **Depends on:** Nothing
   - **Effort:** Small (3-4 days)
```

#### Success Criteria
3-7 specific, measurable, testable conditions per phase. Checklist format.

#### Learning Goals
Questions answered, assumptions validated, feedback obtained.

#### Validation Checkpoints
```markdown
**Date:** [End of Week X]
**Review questions:**
- [Question 1]
- [Question 2]
**Decision criteria:**
- **Proceed:** [What must be true]
- **Pivot:** [What would cause strategy change]
- **Stop:** [What would cause project cancellation]
```

### 5. Dependencies and Sequencing

- **Technical Dependencies:** Feature A requires Feature B
- **Learning Dependencies:** Feature C depends on validating assumption from Feature A
- **External Dependencies:** Features blocked by external factors (with timelines)

### 6. Assumptions and Risks

**Key Assumptions:** What we're assuming is true.

**Sequencing Risks:** What could go wrong with this sequence.

**Mitigation Plans:** Assumption-to-mitigation pairs:
```markdown
- If static analysis <85% accurate -> Add manual linking support
- If development pace <20hr/wk -> Reduce scope or extend timeline
```

### 7. Flexibility and Change

- **Adaptation Triggers:** Events causing roadmap revision
- **Review Cadence:** How often roadmap is revisited (e.g., after each phase + monthly)
- **Change Process:** Identify trigger, assess impact, propose adjustment, review with stakeholders, update document

### 8. Document Control

Version history (reverse chronological) and links to VISION.md, SCOPE.md, feature specs.

## Anti-Patterns

**Missing feature descriptions:** Spec Writer cannot understand features without 1-2 sentence descriptions.

**Vague descriptions:** "Static analysis" or "API work" -- must be specific enough for Spec Writer to start without context.

**Implementation in description:** Describes how feature works -- focus on what feature does (behavior).

**Missing dependencies:** Features list "Nothing" but actually depend on other features -- trace carefully.

**Unmeasurable success criteria:** "Phase complete when done" -- use specific, testable conditions.
