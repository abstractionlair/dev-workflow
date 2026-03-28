# Schema: Scope

The tactical boundaries document that translates strategic vision into concrete deliverables, preventing scope creep and enabling realistic planning.

## Document Header

```markdown
# [Project Name] Scope
```

**Produced by:** Planning Writer (after VISION.md is approved)
**Consumed by:** Planning Writer creating ROADMAP.md, Spec Writer (understands boundaries)
**Lifecycle:** Updated when learnings require scope adjustments (quarterly typical). Versioned in place.

## Required Sections

### Scope Overview

2-3 sentences stating what gets delivered. Concrete and specific -- anyone can understand without reading the full document.

```
GOOD: "DevContext MVP delivers a command-line tool that helps solo developers maintain project context through Git-integrated specifications and living documentation"
BAD:  "A better way to manage projects"
```

### Vision Alignment

```markdown
**Vision Statement:** [Exact copy from VISION.md]
**How This Scope Serves Vision:** [1-2 sentences explaining connection]
```

Vision statement must match VISION.md exactly. No paraphrasing.

### Project Objectives

3-5 concrete, measurable objectives. Each must be achievable within scope and aligned with vision success criteria.

```
GOOD: "Enable spec creation in <2 minutes via CLI"
GOOD: "Context retrieval in <30 seconds via CLI queries"
BAD:  "Improve developer productivity"
```

### In Scope - MVP

**Core Features** (3-10 features at capability level):
```markdown
- **Specification Management:** CLI commands to create, list, edit, and validate specs
- **Context Linking:** Automatic detection of relationships between specs/tests/code
- **Living Documentation:** SYSTEM_MAP.md that stays current with code changes
```

**User Capabilities** (5-10 observable behaviors):
```markdown
- Users can create new feature specs in under 2 minutes by running `ctx spec create`
- Users can find all code related to a spec in under 30 seconds by running `ctx find`
```

Format: "Users can [action] by [method] resulting in [outcome]"

**Technical Requirements:**
```markdown
- Python 3.9+ CLI tool installed via pip
- Works with existing Git repositories (no restructuring required)
```

**Acceptance Criteria** (5-15 testable criteria):
```markdown
- [ ] Creating spec generates Markdown file with all template sections
- [ ] `ctx find` returns results within 1 second
```

### In Scope - Future Phases

**Phase 2 (Post-MVP):** Features for second release, logical extensions of MVP. Brief description per feature.

**Phase 3 and Beyond:** Longer-term additions, less detail. May change based on MVP learnings.

**Deferred Features:** Desired but not prioritized. Acknowledgment prevents "forgotten" feeling.

### Explicitly Out of Scope

**Never in This Project** (minimum 3 exclusions):
```markdown
- General project management (time tracking, resource allocation)
- Communication/chat features (use existing tools)
- Code generation or automated implementation
```

**Not in This Version:** Features possible later but definitely not now.

**External Dependencies:** Things other teams/projects must provide.

### Constraints and Assumptions

**Resource Constraints** (required -- must specify Team, Time, Budget):
```markdown
- **Team:** Solo developer
- **Time:** 15 hours/week for 3 months (180 hours total)
- **Budget:** $0 (open source, no paid services)
```

**Technical Constraints:**
```markdown
- Must work with existing Git repos (no special structure)
- No server/database required (works offline)
```

**Business Constraints:** Compliance, contractual obligations, market timing.

**Assumptions** (with risk and mitigation):
```markdown
- Target users know Git basics
  - Risk: Some users might not
  - Mitigation: Excellent onboarding docs
```

### Success Criteria

**MVP Complete When** (5-10 specific, measurable criteria):
```markdown
1. All acceptance criteria in "In Scope - MVP" are met
2. 5 beta users can complete getting-started guide in under 10 minutes
3. Zero critical bugs reported by beta users
```

**Quality Standards:**
```markdown
- Test coverage >80% for core functionality
- CLI commands respond in <1 second for typical projects
- Zero data loss
```

**Acceptance Process:** Who approves and how.

### Risks and Mitigation

```markdown
### Scope Risks
**Risk:** MVP scope too ambitious for 180 hours
**Probability:** Medium
**Impact:** High (failure to launch)
**Mitigation:** Built-in 20% buffer. If behind at 50% mark, cut Phase 2 features from MVP.

### Execution Risks
**Risk:** Static analysis more complex than expected
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Prototype in week 1. If too complex, simplify to manual links only.
```

### Stakeholder Agreement

- **Key Stakeholders:** Who must approve, their role (decision maker, consulted, informed)
- **Open Questions:** Unresolved scope decisions
- **Change Control:** How scope changes are handled

### Document Control

Version history (version, date, what changed, why) and links to VISION.md, ROADMAP.md.

## Transformation: VISION -> SCOPE

Each vision capability expands into 2-5 concrete deliverables:

| Vision says | Scope provides |
|-------------|---------------|
| "Lightweight specification format" | Core Feature: "CLI commands to create, list, edit, validate specs" |
| (same) | User Capability: "Users can create specs in <2 minutes via `ctx spec create`" |
| (same) | Acceptance Criterion: "Creating spec generates Markdown with all sections" |

## Anti-Patterns

**Feature creep disguised as clarification:** Adding features not in vision under guise of "clarifying" -- additions require vision update first.

**Vague deliverables:** "Better user experience" -- every capability must be concrete and testable.

**Missing "Out of Scope":** Only defining inclusions -- "Never in Scope" is mandatory, minimum 3 items.

**Unrealistic MVP:** Solo dev, 10 hrs/week, wants web + iOS + Android in 6 months -- MVP must fit within constraints.

**No acceptance criteria:** Features listed but no definition of "done" -- every MVP feature needs testable criterion.

**Scope creep:** Continuously adding to "In Scope" without removing -- change control process required; additions go to future phases.

**Treating as specification:** Scope has button labels and API signatures -- scope is capabilities, not specs. Specs come later.
