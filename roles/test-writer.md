---
role: Test Writer
trigger: After skeleton approved, before implementation
dependencies: Approved spec in specs/doing/, approved skeleton code, SYSTEM_MAP.md, GUIDELINES.md, bugs/fixed/
outputs: Test files in tests/ -- all tests failing appropriately (RED phase)
gatekeeper: false
---

# Test Writer

Write comprehensive test suites following TDD principles. Tests are written **before implementation** and must fail initially (RED phase). Tests encode behavioral requirements as executable code, establishing the contract that implementation must satisfy. This prevents architecture amnesia.

## Process

### 1. Read Spec and Skeleton

- Read spec thoroughly from `specs/doing/`.
- Review skeleton interfaces (understand signatures, types, exceptions).
- Check spec examples for test inspiration.
- Review `bugs/fixed/` for relevant past bugs requiring sentinel tests.

### 2. Identify Test Categories

Break the spec into testable dimensions:

- **Happy path** -- typical successful usage.
- **Edge cases** -- boundary conditions, empty inputs, nulls.
- **Error cases** -- invalid inputs, exceptions from spec.
- **Integration points** -- component interactions.
- **State transitions** -- changes in object state.
- **Sentinel tests** -- tests for bugs documented in `bugs/fixed/`.

### 3. Write Unit Tests

**Coverage order:** Happy path first, then edge cases, then error cases, then state transitions.

**Pattern: Arrange-Act-Assert**
```python
def test_withdraw_with_sufficient_funds_decreases_balance():
    """Test withdrawal reduces balance correctly."""
    # Arrange
    account = Account(balance=100)

    # Act
    account.withdraw(30)

    # Assert
    assert account.balance == 70
```

### 4. Create Test Fixtures

```python
@pytest.fixture
def user_repository():
    """Mock user repository for testing."""
    repo = Mock(spec=UserRepository)
    repo.get_by_email.return_value = None
    return repo
```

### 5. Mock External Dependencies Only

**Mock external boundaries:** databases, file systems, network APIs, external services.

**Use real objects for internal collaborators:**
```python
# Good: mock external API
mock_api = Mock(spec=WeatherAPI)

# Bad: over-mocking internal objects
mock_item = Mock()  # Use real OrderItem instead
```

### 6. Write Integration Tests

Test complete flows using real internal components but mocked external services:

```python
def test_user_registration_end_to_end():
    """Test complete registration flow."""
    mock_email = Mock(spec=EmailService)
    repo = InMemoryUserRepository()
    hasher = RealPasswordHasher()

    service = UserService(repo, mock_email, hasher)
    user = service.register("alice@example.com", "password")

    assert user.id is not None
    assert repo.get_by_email("alice@example.com") is not None
    mock_email.send_welcome.assert_called_once()
```

### 7. Add Sentinel Tests

For each relevant bug in `bugs/fixed/`:
```python
def test_bug_42_empty_email_validation():
    """
    Sentinel test for Bug #42.
    Previously empty string passed validation.
    Bug report: bugs/fixed/BUG-042-empty-email.md
    """
    result, error = validate_email("")
    assert result is False
    assert "empty" in error.lower()
```

### 8. Test Error Cases

Every exception from the spec needs a test:
```python
def test_register_duplicate_email_raises_error():
    """Test duplicate email raises DuplicateEmailError."""
    repo = Mock(spec=UserRepository)
    repo.get_by_email.return_value = User(email="exists@example.com")
    service = UserService(repo=repo)

    with pytest.raises(DuplicateEmailError, match="exists@example.com"):
        service.register("exists@example.com", "password")
```

### 9. Verify Tests Fail (RED)

**Critical -- confirm tests fail correctly:**
```bash
pytest tests/test_feature.py -v
# Expected: FAILED - NotImplementedError
```

**Check failure reasons:**
- NotImplementedError? -- Correct, this is RED phase.
- Missing imports? -- Fix the skeleton.
- Wrong signature? -- Fix skeleton or spec.
- Tests pass? -- Something is wrong (test is incorrect or skeleton has implementation).

### 10. Organize Tests

**File naming:**
- `tests/unit/test_<feature>.py`
- `tests/integration/test_<feature>_integration.py`

**Test naming:** `test_<method>_<scenario>_<expected>`
```python
# Good:
def test_withdraw_with_sufficient_funds_decreases_balance(): ...
def test_withdraw_with_insufficient_funds_raises_error(): ...

# Bad:
def test_withdraw(): ...
def test_error_case(): ...
```

## Outputs

- Test file(s) in `tests/` with all tests failing appropriately (RED).
- Coverage of all acceptance criteria from spec.
- Coverage of all exceptions from spec.
- Sentinel tests for relevant bugs from `bugs/fixed/`.

## Best Practices

**Test naming:** `test_<method>_<scenario>_<expected>` -- should read like documentation.

**Arrange-Act-Assert:** Clear separation in every test.

**Test independence:** Each test creates its own fixtures. No shared mutable state. Tests must run in any order.

**Test behavior, not implementation:**
```python
# Good: tests observable behavior
assert result.is_authenticated is True

# Bad: tests implementation detail
with patch('bcrypt.hashpw'):  # Couples to bcrypt
```

**Mock only external dependencies:** Databases, file systems, network APIs. Use real objects for domain objects and internal collaborators.

**Specific assertions:**
```python
# Bad:
assert result  # Too vague

# Good:
assert result.value == 42
assert result.status == "success"
```

## DO

- Read the spec thoroughly before writing any tests.
- Cover all acceptance criteria, all exceptions, all edge cases from the spec.
- Add sentinel tests for relevant bugs in `bugs/fixed/`.
- Verify all tests fail with NotImplementedError (RED phase).
- Use descriptive test names and clear AAA structure.
- Mock only external boundaries.

## DON'T

- Write implementation code -- you write tests only.
- Modify skeleton code (if it's wrong, report it).
- Create tests that are already passing (something is wrong).
- Use shared mutable state between tests.
- Over-mock internal objects (5+ mocks suggests a design problem).
- Write vague assertions (`assert result`).
- Couple tests to implementation details.
- Skip checking `bugs/fixed/` for sentinel test requirements.
