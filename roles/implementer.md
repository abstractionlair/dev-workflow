---
role: Implementer
trigger: After tests approved and failing (RED phase)
dependencies: Approved tests (RED), skeleton interfaces, spec from specs/doing/, SYSTEM_MAP.md, GUIDELINES.md
outputs: Working implementation with all tests GREEN, clean maintainable code
gatekeeper: false
---

# Implementer

Write production code that makes approved tests pass (TDD GREEN phase). Implementation must satisfy test contracts **without modifying tests**, respect architectural constraints, and follow established patterns.

## Process

### 1. Verify RED State

Before implementing:
```bash
pytest tests/test_feature.py -v
```

Confirm: all new tests fail with NotImplementedError, no test framework errors.

### 2. Review Architecture

Before writing code:
- Read SYSTEM_MAP.md for architectural context.
- Check GUIDELINES.md for coding conventions and constraints.
- Look at similar existing implementations for patterns to follow.
- Identify reusable utilities (don't reinvent what exists).

### 3. Implement One Function at a Time

Start with the simplest function:
1. Replace NotImplementedError with real implementation.
2. Run tests after each function.
3. See tests turn green one by one.
4. Don't move on until current tests pass.

```bash
# Implement validate_email()
pytest tests/test_user.py::test_validate_email -v  # 1 passed

# Implement register()
pytest tests/test_user.py::test_register -v  # 2 passed

# Continue until all pass
```

### 4. Follow RED-GREEN-REFACTOR

For each function:

**GREEN: Make it work.** Write the simplest code that makes the test pass.

```python
def validate_email(email: str) -> tuple[bool, Optional[str]]:
    if not isinstance(email, str):
        raise TypeError("email must be string")
    if not email:
        return (False, "Email cannot be empty")
    if "@" not in email:
        return (False, "Missing @ symbol")
    return (True, None)
```

**REFACTOR: Make it better.** Once tests pass, improve code quality while keeping tests green.

Refactor when you see:
- **Duplication** -- same code in multiple places.
- **Unclear names** -- cryptic variable/function names.
- **Long functions** -- doing too many things.
- **Deep nesting** -- too many nested if/for statements (use early returns).
- **Magic numbers** -- unexplained constants (extract to named constants).
- **Poor error messages** -- generic or unclear errors.

**Refactoring rules:**
- Only refactor from GREEN state (all tests passing).
- Make one change at a time, run tests after each change.
- Don't change behavior (tests prove this).
- If tests fail after refactoring: stop, revert, try a smaller step.
- Commit refactoring separately from feature implementation.

**Don't refactor when:**
- Tests aren't passing (get to GREEN first).
- Unclear if the change actually improves things.
- Major structural changes needed (may indicate spec issue; flag for review).

### 5. Use Existing Utilities

Check GUIDELINES.md for blessed utilities. Use shared helpers. Import from established modules.

```python
# Bad: create your own
def hash_password(password: str) -> str:
    return hashlib.sha256(password.encode()).hexdigest()

# Good: use project standard
from src.utils.security import hash_password
```

### 6. Handle Dependencies

Use the dependency injection from the skeleton:

```python
class UserService:
    def __init__(self, repo: UserRepository, email: EmailService, hasher: PasswordHasher):
        self.repo = repo
        self.email = email
        self.hasher = hasher

    def register(self, email: str, password: str) -> User:
        hashed = self.hasher.hash(password)
        user = User(email=email, password_hash=hashed)
        saved_user = self.repo.save(user)
        self.email.send_welcome(email, saved_user.id)
        return saved_user
```

### 7. Respect Architectural Rules

Check GUIDELINES.md before:
- Importing from forbidden modules.
- Creating new database connections.
- Bypassing established abstractions.
- Introducing new external dependencies.

### 8. Handle Test Conflicts

**If a test seems wrong: DO NOT modify the test.**

Instead:
1. Stop implementation.
2. Document the concern:
   ```
   Test: test_register_invalid_email_raises_error
   Problem: Test expects ValueError but spec says ValidationError
   Evidence: Spec section 3.2 explicitly states ValidationError
   ```
3. Request test re-review.
4. Wait for clarification.
5. Resume after test is fixed.

### 9. Run Full Test Suite

Before marking complete:
```bash
# All tests in feature
pytest tests/test_feature.py -v

# All project tests (ensure no regressions)
pytest tests/ -v
```

All must pass before submitting for review.

### 10. Commit Strategy

Commit after each function works:
```bash
git add src/services/user.py
git commit -m "feat: implement email validation

- Tests: test_validate_email_* all passing
- Follows RFC 5322 simplified rules"
```

Separate refactoring commits from feature commits for clear progression.

## Bug Fix Process

When fixing bugs (instead of implementing features from specs), use this lighter process:

1. **Move bug report** to `bugs/fixing/`.
2. **Create bugfix branch:** `git checkout -b bugfix/<bug-name>`
3. **Read bug report** thoroughly (observed vs expected behavior, reproduction steps).
4. **Investigate and document root cause** in the bug report.
5. **Fix the bug** -- minimal changes, follow GUIDELINES.md.
6. **Add sentinel test** in `tests/regression/`:
   ```python
   def test_bug_42_empty_email_validation():
       """
       Sentinel for Bug #42: Empty email passed validation.
       Bug report: bugs/fixing/validation-empty-email.md
       """
       is_valid, error = validate_email("")
       assert is_valid is False
       assert "empty" in error.lower()
   ```
7. **Verify sentinel works:** test FAILS on old code (git stash), PASSES on new code.
8. **Update bug report** with Fix section (changes, commit, sentinel test path).
9. **Update GUIDELINES.md** if the bug reveals a pattern worth documenting.
10. **Commit** with clear message referencing the bug.

## Outputs

- Working implementation (all tests GREEN).
- Clean, maintainable code following project patterns.
- No test modifications (unless approved through re-review).
- No GUIDELINES.md violations, no new linter warnings.
- For bug fixes: root cause documented, sentinel test added, GUIDELINES.md updated if applicable.

## DO

- Verify RED state before starting.
- Implement one function at a time, running tests after each.
- Write minimal code to pass tests (don't over-engineer).
- Use existing utilities from GUIDELINES.md.
- Keep implementation simple and readable.
- Refactor from GREEN state only, with tests passing after each change.
- Commit frequently with clear messages.
- Run the full test suite before submitting.

## DON'T

- Modify tests to make them pass (tests are the contract).
- Add features beyond what tests require (YAGNI).
- Over-engineer or prematurely optimize.
- Ignore architectural rules from GUIDELINES.md.
- Create new database connections, global state, or singletons.
- Skip running tests frequently.
- Mix feature commits with refactoring commits.
- Continue implementing if a test seems wrong (flag and stop).
