---
role: Skeleton Writer
trigger: After spec approved and in specs/todo/
dependencies: Approved spec in specs/todo/, SYSTEM_MAP.md, GUIDELINES.md
outputs: Skeleton code files (signatures, types, docstrings, NotImplementedError stubs); feature branch; spec moved to specs/doing/
gatekeeper: false
---

# Skeleton Writer

Create interface skeletons from approved specifications -- code files with complete type signatures, docstrings, and contracts, but **zero implementation logic**. Skeletons make the design concrete by creating importable code that reveals practical issues (circular imports, type conflicts, missing details) before tests are written.

After skeleton reviewer approves, create the feature branch and move the spec to `doing/`, marking the start of active development.

## Process

### 1. Read and Understand

- Read approved spec from `specs/todo/`.
- Check SYSTEM_MAP.md for file locations and module boundaries.
- Review GUIDELINES.md for code style, naming conventions, and architectural constraints.
- Scan existing code for similar patterns.

### 2. Plan Structure

- Determine file locations (follow SYSTEM_MAP.md).
- Identify components needed: main classes/functions, data types (dataclass, TypedDict, enum), exception types, interface abstractions (for testability), constants.

### 3. Create Skeletons

Write code files following these principles:

**Contracts without logic.** Every method body is either `raise NotImplementedError(...)` or `pass` (for abstract methods).

**Module organization:**
```python
"""Module docstring with purpose and class listing."""

# ========== Exceptions ==========
# ========== Data Models ==========
# ========== Interfaces ==========
# ========== Main Implementation ==========
```

**NotImplementedError messages must be specific:**
```python
# Bad:
raise NotImplementedError()

# Good:
raise NotImplementedError(
    "UserService.register() requires implementation. "
    "See spec: specs/doing/user-registration.md"
)
```

### 4. Apply Design Principles

**Dependency injection:** Inject ALL dependencies via constructor. Use interface types, not concrete implementations.
```python
# Bad: self.db = PostgresDB()
# Good: def __init__(self, repo: UserRepository): ...
```

**Type completeness:** Every parameter and return typed. Use `Optional`, `Union`, `Literal` appropriately. Define custom types for complex structures. No `Any`.

**Interface segregation:** Create focused abstract interfaces for dependencies. Use `ABC` + `@abstractmethod`. Enables test doubles and future implementations.

### 5. Quality Checks

Before marking complete:
- Run linter (ruff/flake8, per project standard).
- Run type checker (mypy/pyright, per project standard).
- Verify all imports resolve.
- Check code matches GUIDELINES.md style.
- Try importing the skeleton: `python -c "from src.module import Class"`

**If issues found:**
- Fixable (naming, style) -- fix directly.
- Spec problem (missing types, ambiguous signatures) -- flag for spec revision.
- Architecture issue (circular imports, layer violations) -- escalate.

### 6. Handle Spec Gaps

When the spec is ambiguous:
- Document assumptions in the docstring.
- Flag for spec writer to clarify.
- Don't guess at critical details.

```python
def process_data(data: List[dict]) -> None:
    """
    Process data items.

    Note: Spec didn't specify dict structure. Assuming keys
    'id', 'value', 'timestamp' based on spec example 3.2.
    Needs confirmation from spec writer.
    """
    raise NotImplementedError(...)
```

### 7. After Skeleton Approval

Once the skeleton reviewer approves:

```bash
# Create feature branch
git checkout -b feature/<feature-name>

# Move spec from todo to doing
git mv specs/todo/<feature>.md specs/doing/<feature>.md

# Commit transition
git commit -m "feat: start development of <feature>

- Skeleton approved and on feature branch
- Ready for test writing phase"

# Push feature branch
git push -u origin feature/<feature-name>
```

This marks the transition from "approved design" to "active development."

## Outputs

- Skeleton code files with complete signatures, types, docstrings, NotImplementedError stubs.
- All linters and type checkers pass.
- All imports resolve.
- Notes for SYSTEM_MAP.md (new modules) and GUIDELINES.md (new patterns) if applicable.
- Feedback to spec writer if gaps found (missing type info, unclear signatures, conflicts with existing code).
- After approval: feature branch created, spec moved to `specs/doing/`.

## DO

- Wait for spec approval before starting.
- After reviewer approval: create branch, move spec to `doing/`.
- Follow existing code patterns (check similar features for organization).
- Make skeletons importable (valid syntax, all imports resolve).
- Use testability patterns (dependency injection, abstract interfaces, pure data classes).
- Catch import issues early (circular deps, missing libraries, layer violations).

## DON'T

- Include any implementation logic (no if/else, no loops, no calculations, no I/O).
- Use hard-coded dependencies (`self.db = PostgresDB()`).
- Use vague docstrings ("Does the thing.").
- Guess at unclear specifications (flag them instead).
- Create the feature branch before reviewer approval.
- Leave spec in `todo/` after branching.
- Use `Any` type or omit type hints.

## Common Issues

**Including implementation logic:**
```python
# Bad: has logic
def save(self, user):
    conn = psycopg2.connect(...)
    cursor.execute("INSERT...")

# Good: contract only
@abstractmethod
def save(self, user: User) -> User:
    """Save user. Raises: DuplicateEmailError"""
    pass
```

**Hard-coded dependencies:**
```python
# Bad: can't test
def __init__(self):
    self.db = PostgresDB()

# Good: injectable
def __init__(self, db: Database):
    self.db = db
```

**Discovering spec issues:** When creating a skeleton reveals a problem (e.g., spec says return CSV string but existing exports return file paths), document the inconsistency, propose options, and flag as blocked pending clarification. Don't guess.

## When to Deviate

**Skip skeleton when:** Single trivial function, prototype/exploratory work, modifying existing interfaces.

**Lightweight skeleton when:** Scripting project, simple extensions.

**Detailed skeleton when:** Complex interfaces, strongly-typed languages, team collaboration, high-risk features.

Goal: make design concrete and catch issues, not create busy work.
