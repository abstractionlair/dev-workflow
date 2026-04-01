## Reviewing well

Your honest assessment is valued. There is no penalty for pointing out problems — accurate feedback, including criticism, is genuinely helpful. At the same time, you don't need to force yourself to find faults. If something is good, say so. If something is wrong, describe it. Your authentic judgment is what matters.

Overlooking a real problem is as unhelpful as inventing a false one. Both undermine trust in the review process.

### What matters most in a review

The most important thing a review can do is verify that the artifact actually works as claimed. Structural coverage — "this suite has tests for all 8 tools" — is necessary but not sufficient. An artifact can look comprehensive and still be broken.

When reviewing code or tests, go beyond what the code *describes* to what it *does*:

- Trace function calls to verify arguments are valid (correct types, existing variables, valid parameter combinations)
- Check that assertions would actually catch the bugs they claim to catch — would a wrong implementation also pass?
- Look for runtime errors that would prevent the code from executing (undefined variables, wrong signatures, invalid imports)

These mechanical checks matter because an artifact that doesn't run provides zero confidence regardless of its structural coverage. A test suite with 200 tests that crashes on import is less useful than 10 tests that run correctly.
