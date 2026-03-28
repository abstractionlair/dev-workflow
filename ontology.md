# Dev-Workflow Ontology

The artifact types, their relationships, state machine, and ownership rules.

## Artifact Relationships

```
VISION.md ──────────────────┐
    |                       |
SCOPE.md                    |  (referenced by all downstream work)
    |                       |
ROADMAP.md                  |
    |                       |
Feature Specs --------------+--- SYSTEM_MAP.md, GUIDELINES.md
    |
Skeletons
    |
Tests <------------------------- bugs/fixed/ (regression tests)
    |
Implementation
```

**Strategic artifacts** (one per project, updated in place with version increments):
- VISION.md -- why the project exists, 2-5 year direction
- SCOPE.md -- what's in/out, boundaries and constraints
- ROADMAP.md -- phased feature sequence

**Living artifacts** (continuously updated):
- SYSTEM_MAP.md -- architecture reference
- GUIDELINES.md -- coding conventions and patterns

**Per-feature artifacts** (flow through state machine):
- Spec -- implementation contract with acceptance criteria
- Skeleton -- interfaces, types, signatures (no logic)
- Tests -- TDD red phase, must fail before implementation
- Implementation -- code that makes tests pass

**Quality artifacts:**
- Review -- evaluation against schema, approval/rejection decision
- Bug report -- reproduction, root cause, fix, sentinel test

## State Machine

### Specs

```
proposed/ --> todo/ --> doing/ --> done/
```

| State | Directory | Meaning | Moved by |
|-------|-----------|---------|----------|
| Draft | `specs/proposed/` | Written, awaiting review | Writer creates here |
| Approved | `specs/todo/` | Review passed, ready to implement | **Spec Reviewer** (gatekeeper) |
| In Progress | `specs/doing/` | Implementation underway | Skeleton Writer (starts work) |
| Complete | `specs/done/` | Implemented and merged | **Impl Reviewer** (gatekeeper) |

### Bugs

```
to_fix/ --> fixing/ --> fixed/
```

| State | Directory | Meaning | Moved by |
|-------|-----------|---------|----------|
| Reported | `bugs/to_fix/` | Needs investigation | Reporter creates here |
| Fixing | `bugs/fixing/` | Investigation in progress | Bug Fixer (self-move) |
| Fixed | `bugs/fixed/` | Fix merged, sentinel test added | **Impl Reviewer** (gatekeeper) |

### RFCs (change requests during implementation)

```
open/ --> approved/ OR rejected/
```

Created when implementation reveals issues with upstream artifacts. Original artifact author decides.

## Ownership Rules

1. **Gatekeepers control forward movement.** Only reviewers move artifacts to the next quality-gated state. Writers and implementers cannot self-approve.
2. **One artifact, one state.** Use `git mv` (not `cp`) for atomic transitions.
3. **Review records are immutable.** Once created, never moved or deleted.
4. **Terminal states are permanent.** `done/`, `fixed/`, `approved/`, `rejected/` are final.

## Directory Layout

```
project/
  .workflow/
    config.json                  # Project-level workflow settings
  docs/
    VISION.md
    SCOPE.md
    ROADMAP.md
    SYSTEM_MAP.md
    GUIDELINES.md
  specs/
    proposed/                    # Awaiting review
    todo/                        # Approved, not started
    doing/                       # In progress (on feature branch)
    done/                        # Completed
  bugs/
    to_fix/
    fixing/
    fixed/
  rfcs/
    open/
    approved/
    rejected/
  reviews/
    specs/
    implementations/
    bugs/
  src/                           # Implementation code
  tests/                         # Test code
    regression/                  # Sentinel tests for fixed bugs
```

## Branching Strategy

- **Main branch**: planning docs, specs in `proposed/` and `todo/`, living docs
- **Feature branches**: created when spec moves to `doing/`, contains skeleton + tests + implementation
- **Merge trigger**: implementation reviewer approves, merges feature branch, moves spec to `done/`

## Planning Doc Updates

Strategic docs don't move between directories. They're versioned in place:
- **Minor** (v1.0 -> v1.1): clarifications, small additions
- **Major** (v1.0 -> v2.0): strategic changes, require re-review

## Role Catalog

| Role | Layer | Creates | Gates | Default Harness |
|------|-------|---------|-------|-----------------|
| Planner | Strategic | Vision, Scope, Roadmap | -- | claude |
| Spec Writer | Design | Specs | -- | claude |
| Macro Reviewer | Quality | Reviews | All state transitions | gemini (docs), opencode/gpt-5.4 (code) |
| Micro Reviewer | Quality | Reviews | -- (advisory only) | opencode (open models) |
| Skeleton Writer | Code | Interfaces, types | -- | claude |
| Test Writer | Code | Test suites (RED) | -- | claude |
| Implementer | Code | Working code (GREEN) | -- | claude |

## Review Matrix

| Artifact | Writer | Macro Review | Micro Review |
|----------|--------|-------------|-------------|
| Vision | planner | gemini | -- |
| Scope | planner | gemini | -- |
| Roadmap | planner | gemini | -- |
| Spec | spec-writer | gemini | -- |
| Skeleton | skeleton-writer | opencode/gpt-5.4 | opencode/open model |
| Tests | test-writer | opencode/gpt-5.4 | opencode/open model |
| Implementation | implementer | opencode/gpt-5.4 | opencode/open model |
