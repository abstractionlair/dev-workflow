# Dev-Workflow — Open Gaps

Captured 2026-05-10 while comparing current state against predecessors [`dev_workflow_meta`](https://github.com/abstractionlair/dev_workflow_meta) and [`MultiModelCLIEmail`](https://github.com/abstractionlair/MultiModelCLIEmail). Items below are real holes, not nice-to-haves: each is a place where the ontology promises behavior the project doesn't actually deliver.

## Ghost features (structure exists, behavior doesn't)

### RFC process

`ontology.md` defines an `rfcs/{open,approved,rejected}/` state machine for "change requests during implementation," but there is no role file, no skill, and no documented protocol for when to file one, who reviews it, or how it propagates back to the upstream artifact.

`dev_workflow_meta` had `FeedbackLoops.md` (strategic re-planning) and `RFC.md` (tactical mid-implementation change requests) as separate processes. Useful prior art if we want to fill this in rather than remove it.

Decision needed: implement, or strip from ontology.

### Bug workflow

`schemas/bug-report.md` exists. `ontology.md` defines `bugs/{to_fix,fixing,fixed}/` with the Impl Reviewer as gatekeeper. But there is no `bug-fixer.md` role file, no `/workflow-bug` skill, no documented sentinel-test protocol.

`dev_workflow_meta` had `role-bug-recorder.md` covering the bug → reproduction → root cause → fix → sentinel test cycle.

Decision needed: implement, or strip from ontology.

## Documentation holes

### No end-to-end walkthrough

`dev_workflow_meta` had `WorkflowExample.md`. Current project has none. For something meant to be imported into other projects, a "here's a full feature traversal from vision to done" doc is the single biggest onboarding gap.

### Living docs (SYSTEM_MAP.md, GUIDELINES.md) referenced but not specified

`ontology.md` lists these as living artifacts but no schema defines what they should contain, when they're updated, or who owns them. `dev_workflow_meta` had full schemas for both (~1,460 + ~1,625 lines).

Decision needed: write schemas, or downgrade from "living artifact" to "project decides."

## Predecessors not flagged

Done as of 2026-05-10 — README.md now has a History section naming both predecessors.

## Planned: three explicit aims for reviews and tests

Captured 2026-05-10 just before the coordinator went offline. The intent is to make reviews and tests address three distinct aims, in parallel — possibly as explicit steps within each. Both reviewer role files and test-writer role file should incorporate this. WORKFLOW.md should be updated to match.

**Aim 1 — Did we build what we set out to build, and did we build it correctly?**

System-level / end-to-end correctness. Explicit reviewer attention and explicit tests for each. Examples the coordinator gave:

- Did we actually write the new computation?
- Do the right numbers come out for a good set of test inputs?
- Did we wire it in correctly?
- If we added a config option to turn it on, does it in fact turn it on?

This is the highest-stakes aim: the feature does the thing the spec asked for, observable from outside.

**Aim 2 — What could go wrong?**

The coordinator: "what could go wrong feels close. May even be it." Marked partly placeholder because they weren't certain there wasn't a different concept they had forgotten.

The aim covers failure modes, edge cases, error paths — what happens when inputs are bad, services are down, dependencies fail, assumptions are violated, the unexpected happens. The gap between "passes happy-path tests" and "is actually correct under realistic conditions."

Candidates from the original placeholder, in case the missing concept was distinct from this one:

- *Module-level structural integrity.* Do the abstractions and interfaces compose, are responsibilities placed correctly. Distinct from Aim 1's "wired in correctly," which is about integration with existing code.
- *Non-regression / impact on other code.* Did this change break anything outside the feature.
- *Maintainability / smell.* Will future-me be able to read and modify this in six months.

**Aim 3 — Low-level, line-by-line.**

Local correctness in the small. Examples the coordinator gave:

- Off-by-one errors.
- Edge cases.
- Wrong parameters to a library call.
- Are (almost) all interfaces typed?
- Do types all mesh?

**Possibly an additional aim — Code and architectural quality**

Captured 2026-05-10 while the coordinator was travelling on a spotty connection. They said: "May not be number 2 but should be captured." Position unsettled — could be a fourth aim, could fold under one of the others, could be its own thing.

The aim: overall code quality, plus architectural quality where applicable.

- Is the code concise and idiomatic?
- Are we duplicating functionality?
- Is the division of capabilities orthogonal — are we carving nature at the joints?

This is distinct from "what could go wrong" (which is about behavior under stress) and from line-by-line bugs. It is about whether the code is *well-made* — readable, non-redundant, structurally sound.

**Where all of this lands when filled in**

- `WORKFLOW.md` — "Tests are read carefully" and "Reviews are how I find out what I don't know" both grow to make the aims explicit.
- `roles/macro-reviewer.md` and `roles/micro-reviewer.md` — review process incorporates the aims as separate review dimensions, with guidance on what each aim looks like in practice.
- `roles/test-writer.md` — tests are organized to cover the aims explicitly rather than as a flat list.
- `schemas/review.md` — review document structure surfaces the aims so reviewers can't conflate them.

## Planned: explicit "make it good" step in implementation

Captured 2026-05-10 alongside the code-quality aim — the coordinator was reminded of advice they liked: when doing TDD or similar, there should be a step after "make it work" to "make it good."

The implementation phase should explicitly include a refactor step. Once tests pass, take the time to clean up structure, names, duplication, and any architectural awkwardness that surfaced while making it work. The classic Beck cadence: red → green → refactor.

This is the same code-quality concern as the additional aim above, landing at a different point in the workflow — at write-time rather than review-time.

Where this lands:
- `roles/implementer.md` — the implementer's process explicitly includes a refactor pass after tests are green.
- `WORKFLOW.md` — the personal doc may mention this where it talks about implementation, though the personal doc's emphasis is on review-side trust mechanisms; this might fit better in the project-side role file alone.

## Structural gap: every gate is sandboxed, so nothing verifies against reality

Captured 2026-08-02 from the `computer-use-mcp` build, which ran the full stage sequence with review gates between every stage and still shipped a defect that made the product unusable in ordinary use.

**What happened.** The spec pinned an exact `xdotool` invocation. Two independently written test suites asserted that argv. Six model reviews across six panels read that argv against the spec. Everything passed: 195 tests green, ruff and mypy clean, reviewers explicitly confirming interface fidelity tool by tool. The invocation hangs forever against a real X server whenever the pointer is already at the target coordinate — so clicking the same spot twice failed, which is not an edge case. It was found by a live click, minutes after the last gate closed.

**Why nothing caught it.** The suites mock the subprocess layer, because a test that needs a display is a test that cannot run in CI. A mocked test can only assert that *the argv you intended is the argv you built*; it cannot tell you the argv is wrong. And every reviewer read the same argv against a spec that specified the same wrong thing, from inside sandboxes that could not execute it — the codex sandbox cannot open the host X socket at all.

This is not six reviewers being lazy. It is six reviewers sharing one blind spot. The workflow's trust story rests on diversity of perspective, and diversity of perspective buys independence of *opinion*; it buys no independence from *reality*. Adding a seventh model does not thin this failure mode, because the property they share is not their training data — it is that none of them can run the thing.

The same gap appeared in the human-facing half of the same project, which suggests it is general rather than a quirk of one defect: the remote-access path was designed, documented, and marked complete without the transport ever being exercised. Three separate things blocked it on first real use, one of which was an sshd policy that had forbidden the entire mechanism for weeks. **Documenting a path is not testing a path.**

**Proposed fix — a live-verification stage that no model can satisfy.** After implementation review, before an artifact may move to `done/`, a stage whose exit criterion is running the real thing against the real world and reporting measured values, not assertions. Its distinguishing property is that it must be executed by whoever has un-sandboxed access — in practice the human, or the routing session — because a sandboxed reviewer *structurally cannot* discharge it. Notes toward a design:

- The stage's output is numbers and observed strings, not a pass/fail claim: actual dimensions, actual returned text, actual exit codes. On this project the useful artifact was "screenshot over MCP: image/png 1920x1080", "out-of-bounds click -> Error: Coordinates (5000, 5000) are outside display bounds (1920x1080)".
- It should name, in advance and in the spec, which behaviours can *only* be verified live. Any behaviour whose test must mock an external binary, a network, a display, or a clock is a candidate. That list is writable at spec time and is a natural companion to the failure-modes section already planned above.
- It needs an explicit answer for what happens when live verification contradicts a passing test, since that is the interesting case: here the fix required changing the spec, both suites, and the code together.
- Scale it like the rest of the workflow — a throwaway script for a small feature, something durable for a subsystem.

**Where this lands:**
- `ontology.md` — the artifact/state flow gains a stage, or `done/` gains an entry condition.
- A new role file, or an extension of `roles/macro-reviewer.md` with an explicit non-delegable step.
- `schemas/spec.md` — a section naming the behaviours that can only be verified live.
- `WORKFLOW.md` — this belongs on Scott's "Additions I want to make" list, in his words not mine. Related items already there: "Failure-mode documentation alongside features" (the natural companion) and "Incident learning loop" (this entry is an instance of one).

**Supporting evidence from the same build, worth keeping when this is designed:**
- *Two independently written test suites are a working instrument.* Both were written from the same spec by different models that could not see each other's work. When six of one suite's tests failed, the implementer refused to edit them and filed a disputes document quoting each test against its spec clause; all six were test defects, and their author fixed them without weakening one. The structure made "test bug or code bug?" decidable, and made bending a test structurally unavailable to the implementer. This is the test-side analogue of "Independent re-implementation for the highest-stakes pieces" in `WORKFLOW.md`, and it was much cheaper than that sounds.
- *Never prime a verifier.* A fix-verification pass given the implementer's own account of what changed returned an unqualified all-clear on all six fixes. A cold reviewer, told only what code to examine, found a real resource leak in the same code. A reviewer handed the change list checks whether the change is present, not whether it is sufficient.

Decision needed: implement as a stage, or state explicitly that live verification is out of scope for the workflow and belongs to the operator — but the current silence reads as coverage that does not exist.

## Notes for future-me

- The 3-model review of the v1.0 enhanced-review-workflow spec on 2026-04-01 said unanimously that it read as a design document, not a behavioral contract — missing interface signatures, Given/When/Then scenarios, error-path acceptance criteria, schema-required sections (Feature ID, References, Scenarios), and one internal contradiction (spec said reviewer follow-ups must not reveal which model wrote which review, while another section labeled them `Reviewer [model]`). v2.0 fixed it. The same critique could land on parts of the current project itself: ontology.md describes RFC and bug machinery in the language of "this is how it works" without the contracts that would let someone actually implement it.
