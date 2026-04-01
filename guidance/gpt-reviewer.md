## Reviewing well

The purpose of the review is to assess the degree of justified confidence the artifact provides. Not binary pass/fail — degree.

A suite that correctly covers 90% of the spec with well-constructed tests is materially different from a suite with broken tests that would fail on a correct implementation. Both have gaps, but they are not the same kind of gap. Your verdict should reflect this distinction.

When assessing confidence:

- Tests that are broken (wrong arguments, undefined variables, incorrect assumptions about API behavior) actively undermine confidence — they are worse than missing tests, because they produce false signal.
- Missing coverage of a spec behavior reduces confidence proportionally to the importance of that behavior. Missing a core happy path matters more than missing an edge case.
- A test that asserts too little (e.g., `assert result` instead of checking specific fields) provides some confidence that the code doesn't crash, but little confidence that it's correct. Note this, but weigh it appropriately.

Your judgment about severity and weighting is what makes the review valuable. Two reviewers who find the same gaps but weigh them differently are both contributing useful signal.

### The test suite's purpose

The test suite exists to be a correctness gate: when all tests pass (GREEN), we should be justified in believing the implementation is correct and complete. That is the primary concern.

RED-phase behavior (how tests fail against an unimplemented skeleton) is a secondary, process-level concern. Specifically:

- **Fixture cascading is not a blocking issue.** If a test for `goto` errors during setup because its fixture calls `capture()` and `capture()` is unimplemented, that test will naturally progress from ERROR to FAIL to PASS as implementation proceeds. This is temporary and self-resolving. Requiring fixtures to seed state directly to achieve RED purity can actually *undermine* correctness confidence — hand-crafted state bypasses the integration between methods (e.g., `capture` setting provenance_parent, moving the cursor, updating breadcrumbs), which means the GREEN-phase test exercises less of the real system.
- **Correctness concerns are blocking; process concerns are not.** A test that would pass with a wrong implementation is a correctness problem — blocking. A test that errors in setup during RED instead of failing at its target method is a process inconvenience — note it, but do not block on it.
- **Tests that fight the skeleton are a real problem.** A test that rejects `NotImplementedError` during RED — whether explicitly (`pytest.fail`) or implicitly (via regex match against stub messages) — produces false signal. That is worth flagging. But a test that simply cascades through an unimplemented fixture is not fighting the skeleton — it's waiting for its dependencies.
- **Distinguish spec-contracted from unspecified behavior.** Tests for behaviors the spec explicitly contracts (defined exceptions, stated invariants, documented postconditions) belong in RED — they correctly signal "this isn't implemented yet." Tests for behaviors the spec does *not* contract (e.g., what error to raise for an invalid session token when no exception type is defined) are better deferred to implementation. During RED review, flag unspecified-behavior tests as "defer to implementation" rather than blocking on them, and note the gap so the implementer knows to address it.
