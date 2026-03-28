# Schema: Vision

The strategic foundation document that guides all downstream planning and implementation decisions.

## Document Header

```markdown
# [Project Name] Vision
```

**Lifecycle:** Created at project inception. Updated quarterly or when major learnings invalidate assumptions. Versioned in place (minor: v1.0->v1.1 for clarifications; major: v1.0->v2.0 for strategic changes).

**Produced by:** Planning Writer
**Consumed by:** Reviewer (validates quality), Planning Writer creating SCOPE.md and ROADMAP.md

## Required Sections

### Vision Statement

1-2 sentences that pass the elevator test.

- Must mention target user (not just product)
- Must describe outcome/benefit (not features)
- Must be solution-agnostic (allows strategic pivots)
- Guides what NOT to build

```
GOOD: "Help solo developers maintain project context across planning and execution without documentation overhead"
BAD:  "Build the best project management tool with AI and collaboration" (feature list)
BAD:  "Revolutionize the industry" (too vague)
```

### Problem Statement

#### Current State
- Specific problem description (not generic)
- Who experiences this problem (link to Target Users)
- Quantified pain where possible (hours wasted, error rates)
- Root causes (not just symptoms)

#### Desired Future State
- Concrete, measurable improvement
- How users' lives will be different
- How to know when achieved (links to Success Criteria)

### Target Users

#### Primary Persona (required)
```markdown
**Name/Role:** [e.g., "Solo Developer Sarah"]
**Demographics:** [Just enough to target]
**Current Behavior:** [How they solve this problem today]
**Jobs-to-be-Done:** [What they're trying to accomplish]
**Pain Points:** [Specific frustrations]
**Decision Criteria:** [What makes them choose solutions]
```

Secondary personas optional, same structure with lighter detail.

### Value Proposition

#### Core Benefit
Primary outcome improvement -- focus on "how lives are improved" not "what product does."

#### Differentiation
```markdown
**Unlike** [primary competitive alternative],
**our product** [statement of primary differentiation]

**Why users will choose this:**
- [Unique advantage 1]
- [Unique advantage 2]

**Why advantage is sustainable:**
[What makes this defensible]
```

### Product Scope

**In Scope (MVP):** 3-7 core features delivering primary value, must-have capabilities, technical requirements. Capability level, not detailed specs.

**Future Scope:** Features for later versions, growth considerations. High-level, may change based on MVP learnings.

**Never in Scope:** Explicit exclusions -- what we deliberately will not do. Prevents scope creep by making exclusions explicit.

### Success Criteria

#### Key Metrics
3-5 specific, measurable metrics. Each must have:
- Definition (what it measures)
- Current baseline
- 6-month goal
- 1-year goal

Metrics should measure value delivered, not vanity.

#### Counter-Metrics (Guardrails)
2-3 metrics ensuring quality isn't sacrificed for growth.

#### Timeline Milestones
- **6 Months:** [Definition of success]
- **1 Year:** [Aspirational goal]
- **3 Years:** [Vision achievement markers]

### Technical Approach

- **Technology Stack:** Frameworks, platforms, key dependencies with rationale
- **Architecture Principles:** Core decisions, constraints, trade-offs
- **Known Risks:** Technical challenges with mitigation approaches

### Assumptions and Constraints

- **Market Assumptions:** User behavior predictions, competitive scenarios
- **Technical Assumptions:** Feasibility, available tools, required skills
- **Resource Constraints:** Time (hours/week), budget, team size/skills

### Open Questions

- Unresolved decisions requiring resolution
- Areas needing research
- Hypotheses needing validation
- Prioritized by which questions block progress

## Anti-Patterns

**Feature list as vision:** "Build platform with X, Y, Z" -- focus on customer outcomes, not features.

**Mission confusion:** "Make the world better" -- must be product-specific, time-bound, measurable.

**Solution lock-in:** "Build mobile app for..." -- use outcome language: "Enable users to... anywhere."

**No boundaries:** Everything in scope, nothing excluded -- "Never in Scope" section is mandatory.

**Vanity metrics:** "1M users" -- use value metrics: "Users reduce task time by 80%."

**Treating as immutable:** Never updating despite learnings -- quarterly reviews, update based on evidence.

**Updating too frequently:** Vision changes monthly -- vision should be stable; strategy can change.
