---
name: workflow-build
description: Build from an approved spec — skeleton interfaces, TDD tests, then implementation. Use when a spec is approved and ready to implement.
---

# Workflow Build

Implement an approved spec through three phases: skeleton, tests, implementation.

## Process

### 1. Load context

```
Read ~/deploy/dev-workflow/preamble.md
```

Follow the preamble instructions to scan project state.

### 2. Find approved spec

Check `specs/todo/` for approved specs ready to implement. If the user specified a feature, find that spec. If multiple specs are ready, ask which one.

If no specs in `todo/`, check:
- `specs/proposed/` — "This spec needs review first. Run `/workflow-review`."
- No specs — "No specs found. Run `/workflow-spec` to create one."

### 3. Determine current build phase

Check what already exists for this feature:
- **No skeleton exists**: Start with Skeleton phase
- **Skeleton exists, no tests**: Start with Test phase
- **Tests exist, no implementation**: Start with Implementation phase
- **Everything exists**: "Feature appears complete. Run `/workflow-review` for implementation review."

For a spec moving from `todo/` to `doing/`, create a feature branch:
```bash
git checkout -b feature/<feature-name>
git mv specs/todo/<name>.md specs/doing/<name>.md
git commit -m "Start implementing: <feature-name>"
```

### 4. Skeleton phase

```
Read ~/deploy/dev-workflow/roles/skeleton-writer.md
Read specs/doing/<name>.md
```

Follow the skeleton-writer role. Key behaviors:
- Create interfaces, types, function signatures from the spec
- Use `NotImplementedError` / `TODO` stubs — no logic
- Follow existing project conventions (read GUIDELINES.md / SYSTEM_MAP.md if they exist)
- Place code in the project's source directory

### 5. Test phase

```
Read ~/deploy/dev-workflow/roles/test-writer.md
Read specs/doing/<name>.md
```

Follow the test-writer role. This integrates with the existing `/test-driven-development` skill. Key behaviors:
- Write tests that exercise all acceptance criteria from the spec
- Tests reference the skeleton interfaces
- **Tests MUST fail** (TDD red phase) — verify this by running them
- Do NOT write implementation code
- Place tests in the project's test directory

### 6. Implementation phase

```
Read ~/deploy/dev-workflow/roles/implementer.md
Read specs/doing/<name>.md
```

Follow the implementer role. Key behaviors:
- Write minimal code to make tests pass (TDD green phase)
- **Do NOT modify tests** — the tests are the contract
- **Do NOT add features** beyond what tests require
- Run full test suite after implementation
- Refactor only after all tests pass

### 7. After implementation

- Run the full test suite to confirm everything passes
- Suggest review: "Implementation complete. Run `/workflow-review` for implementation review, or use a different model terminal for cross-model review."
- The implementation reviewer will merge the feature branch and move the spec to `done/` on approval.

Report status: **DONE** with which phases were completed and test results.
