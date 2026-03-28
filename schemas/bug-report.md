# Schema: Bug Report

Documents defects in production code: incorrect existing behavior, root cause, fix, and sentinel test to prevent recurrence.

## File Metadata

**Filename:** `<component>-<brief-description>.md` (kebab-case)
**Location:** `bugs/to_fix/` -> `bugs/fixing/` -> `bugs/fixed/`

```yaml
---
reported: YYYY-MM-DD
reported_by: human | <agent-name>
severity: critical | high | medium | low
status: to_fix | fixing | fixed
fixed: YYYY-MM-DD  # Added when moved to fixed
---
```

**Severity guidelines:**
- **Critical:** System unusable, data loss, security breach
- **High:** Major feature broken, significant user impact
- **Medium:** Feature degraded, workaround exists
- **Low:** Minor issue, cosmetic, edge case

## Required Sections (for reporting)

### Title

Pattern: `[Component] Brief description`

```
[Validation] Empty email passes validation
[Auth] SQL injection in email lookup
[Session] Race condition on concurrent updates
```

### Observed Behavior

What currently happens (incorrect). Must include specific symptoms, error messages if any, when/where it occurs.

### Expected Behavior

What should happen instead. Must be clear, specific, and testable.

### Steps to Reproduce

Numbered steps to trigger the bug.

```markdown
## Steps to Reproduce
1. Call `validate_email("")`
2. Observe returns `(True, None)`
3. Attempt to save user with empty email
4. Database raises constraint violation
```

### Impact

Severity rating plus user impact, system impact, and security implications if any.

## Sections Added During Fix

### Root Cause

Added by Implementer after investigation. Explains what caused the bug at the code level.

### Fix

Added by Implementer after fixing. Must include:
- Summary of code changes
- Commit reference
- Sentinel test location
- GUIDELINES.md updates (if any)
- Verification that sentinel test fails on old code and passes on new code

## Complete Example

```markdown
---
reported: 2025-10-23
reported_by: human
severity: high
status: fixed
fixed: 2025-10-24
---

# [Validation] Empty Email Passes Validation

## Observed Behavior
Empty string passes email validation and causes database constraint violation
downstream. `validate_email("")` returns `(True, None)` instead of rejecting.

## Expected Behavior
Empty string rejected: `(False, "Email cannot be empty")`.

## Steps to Reproduce
1. Call `validate_email("")`
2. Observe returns `(True, None)`
3. Attempt to save user with empty email
4. Database raises `NOT NULL constraint failed: users.email`

## Impact
**Severity:** High
- Users can submit forms with empty emails
- Causes 500 errors at database layer
- Affects registration, profile update, and email change flows

## Root Cause
Email validation checked for @-symbol before checking if string was empty.
Empty string has no @-symbol but bypassed the format check and returned
success. The function assumed any string without @ was "invalid format" but
didn't distinguish between empty input and malformed input.

## Fix
Added empty/null check as first validation step before any format checks.

**Changes:**
- `src/utils/validation.py`: Added empty string check at function entry
- `tests/regression/test_validation_empty_email.py`: Sentinel test added
- `GUIDELINES.md`: Added "Validate empty/null inputs first" pattern

**Commit:** abc123def456

**Sentinel test:** `tests/regression/test_validation_empty_email.py`

**Verification:**
- Sentinel test fails on old code
- Sentinel test passes on fixed code
- All other tests still pass
```

## State Transitions

| Transition | Who | Action |
|-----------|-----|--------|
| to_fix -> fixing | Implementer | Moves file, creates bugfix branch, begins investigation |
| fixing -> fixed | Reviewer (gatekeeper) | Moves file, updates status/date, merges to main |

## Anti-Patterns

**Vague description:** "Email validation is broken" -- must state exactly what input produces what incorrect output.

**Missing reproduction steps:** "Sometimes the validation crashes" -- must provide deterministic steps to trigger.

**No impact assessment:** "This is bad and should be fixed" -- must include severity and scope of affected functionality.

**Premature solutions:** Prescribing a solution in the bug report -- describe observed vs expected behavior; let Implementer determine solution.

**Multiple bugs in one report:** "Various validation issues" -- one bug report per issue, each needs separate investigation and testing.
