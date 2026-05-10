# Work-Plan

Prescriptive companion to WORKFLOW.md. WORKFLOW.md describes the pattern and reasons about it; this doc says what to actually do. Written so I can open it in a work Claude Code session and use it as a working plan — and so the plan doesn't only live in my head.

Generic across features by design. The point is the standing shape of the work, not the next feature in particular. Updated over time.

Two things land here together:
- **Ramp-up.** Moving the pattern from "I do most of this informally" to "I do all of this on purpose."
- **Risk additions.** The seven additions enumerated in WORKFLOW.md, woven in where they bite (per-feature) or pulled out as standing practices (everything else).

Status: in motion; expect revision.

## Per-feature pipeline

Each feature passes through phases. Each phase has: what I do, where the risk additions land, and which roles/skills exist (purpose only — implementation TBD, see roles inventory below).

### Spec

What I do: sit with a model in sustained bidirectional conversation until the spec captures intent, behaviour, edge cases, error paths, and what would falsify "this is working." The model challenges; I refine. I keep going until the model stops finding gaps and I stop finding things I haven't said clearly.

Risk addition lands here: **failure-mode documentation alongside features.** The spec captures not just the happy path but the ways it can go wrong — explicit, named, in the doc. The test-writer reads this; the reviewer reads this.

Roles/skills: spec-helper (Socratic dialogue), spec-writer (drafts the spec).

Output: spec doc in version control. Reviewed before code is written against it.

### Spec review

What I do: get reviews against the four aims from a macro reviewer and a micro reviewer, run in independent sessions. At work, frontier is Claude-only, so independence comes from harness + prompt + tier rather than vendor at the frontier. (Top-tier OS at the micro tier adds a second vendor below the frontier.)

Aims at this stage:
- *Did we set out to build the right thing?* (macro)
- *What could go wrong?* (macro — applied to the spec: under what real conditions does what the spec describes fail to be enough?)
- *Line-by-line correctness in the spec* (micro — internal consistency, terminology, schema conformance)
- *Quality of the spec itself* (micro — is it clear, non-redundant, well-carved?)

I revise until the reviews clear. I do not advance past a review I disagree with by ignoring it — I either change the spec or write back to the review explaining why I'm rejecting it. That note becomes part of the spec history.

Roles/skills: reviewer (parameterised by tier and aim).

### Skeleton

What I do: collaboratively design interfaces, type signatures, and module structure. Architectural decisions land here. If a new architectural decision is significant, the architecture documentation gets updated in the same change.

Risk addition: the skeleton is where the code-quality aim takes its first bite. Are responsibilities placed correctly? Are interfaces orthogonal? This is cheaper to fix here than after implementation.

Roles/skills: skeleton-writer.

### Skeleton review

What I do: macro review focused on the code-quality aim and the "did we wire it in correctly" piece of the correctness aim. Cheaper to redo a skeleton than a finished implementation; I push hard at this stage.

### Tests

What I do: write tests organised explicitly around the four aims, not as a flat list. Each aim's tests are identifiable as such — by directory, by name, by section. The failure-mode tests draw on what the spec said could go wrong.

This is the TDD red phase: tests fail because the code does not exist.

Roles/skills: test-writer.

### Test review

What I do: review tests for coverage against the aims, and for test quality. Tests are code too. The test review frequently surfaces gaps in the spec — at which point I go back to spec, not forward to implementation. Stage advancement is gated on this.

### Implementation

Two sub-steps, both mandatory:

**Make it work.** TDD green phase. Tests pass. Get there as directly as the structure allows.

**Make it good.** Refactor phase. Clean up structure, names, duplication, architectural awkwardness that surfaced while making it work. Beck's red→green→refactor cadence. The refactor step is not optional — skipping it accumulates the debt the workflow exists to prevent.

Roles/skills: implementer (handles both steps explicitly).

### Implementation review

What I do: full four-aim review at both tiers. Macro tier handles end-to-end correctness, what-could-go-wrong, and architecture/code-quality at the system level. Micro tier handles line-by-line correctness and code-quality at the local level.

Diversity stack at work, in compounding layers:
- *Cross-tier:* frontier Claude (macro) and top-tier OS (micro). Different capability classes catch different things.
- *Cross-harness:* Claude Code, internal harness, OpenCode if useful. Different system prompts and tool sets produce different reasoning trajectories.
- *Cross-prompt:* structured questions checked against external standards, not adversarial "find what's wrong" prompts. Adversarial framing produces confabulated criticism — models read between the lines and invent issues to justify the framing.
- *Cross-vendor:* available at the OS tier (e.g. Qwen vs Llama). Not available at the frontier within work constraints.

The find/verify asymmetry kicks in here: models find candidate issues frequently faster than I would; I verify or dismiss each one. When the model is stuck and I'm not, the asymmetry flips — that's a useful signal, not an exception.

### User guides and demos

What I do: after implementation passes review, write a user guide and a runnable demo for the new functionality. Audiences: me coming back months later; future model instances picking up the code in a fresh session, whether as reviewers, extenders, or debuggers; colleagues in several roles — reviewers of the code, users of the new functionality, future developers who'll extend or modify it.

Roles/skills: guide-writer (may fold into spec-helper-style dialogue rather than being a separate role).

### Architecture documentation update

What I do: update the relevant architecture documents to reflect the new state. If the change is architecturally significant and the doc was already due for a lift, this is when it happens.

Aspiration is publication-grade. Honest current state is that some of my architecture docs are notes-quality — good enough for me and a returning model session, but I wouldn't point a colleague at them without extra diligence. The standing practice below addresses this.

Roles/skills: architecture-docs writer.

## Standing practices

Run on their own cadence, not per-feature.

### Scheduled code reading

What I do: read recent diffs in features I implemented, on a cadence, away from the moment of writing. Keeps my mental model honest. Surfaces drift between what I think the code does and what it actually does. Surfaces follow-up I missed.

Cadence to set after starting. Probably weekly. Adjust based on volume.

### Periodic drift checks

What I do: run a structured comparison between artefacts and code on a cadence — architecture docs vs current code, finished specs vs implementation, user guides vs actual behaviour. Drift is invisible until I look for it; the cadence makes the looking happen.

Cadence: every N features, or monthly, whichever comes first. Adjust.

Roles/skills: drift-checker.

### Accountability checkpoints

What I do: explicit moments where I sign off — "I understand this and stand behind it" — that are distinct from "reviews passed." Before merge of anything non-trivial; before release; before deploys with elevated risk. The asymmetry between human accountability and AI accountability is honoured explicitly, not implicitly. If I cannot sign off in those terms, the work is not ready.

### Incident learning loop

What I do: after an incident, a post-mortem that captures cause, fix, and a sentinel test that would have caught it. Findings feed back into spec/test/review process for future features in adjacent areas.

This is also where the bug workflow lives: bug → reproduction → root cause → fix → sentinel test.

Roles/skills: bug-fixer / incident-learner.

### Publication-grade architecture docs

What I do: lift one architecture doc per quarter (approximate cadence) from notes-quality to a bar where I'd point a colleague at it. Not every doc at once. Incremental.

### Independent re-implementation

What I do: for the highest-stakes pieces, implement twice — by different models, in different sessions, against the same spec — and compare. Rare. Reserved for pieces where a subtle bug would be expensive to recover from.

## Roles and skills inventory

Roles the workflow uses, with one-line purposes. Whether they become formal role files at work (as in this project) is a separate decision — they may live as inline guidance, structured prompts, or named conventions instead.

- **spec-helper** — Socratic dialogue to surface spec gaps before drafting.
- **spec-writer** — Produces specs from conversation plus reference material.
- **skeleton-writer** — Produces interfaces, type signatures, module structure.
- **test-writer** — Produces tests organised around the four aims.
- **implementer** — TDD green plus refactor; makes tests pass, then makes the code good.
- **reviewer (macro)** — End-to-end correctness, what-could-go-wrong, architecture/code-quality at the system level. Frontier capability.
- **reviewer (micro)** — Line-by-line correctness, code-quality at the local level. Top-tier OS capability sufficient.
- **guide-writer** — User-facing documentation, demos, examples.
- **architecture-docs writer** — System-level documentation; publication-grade aspiration.
- **drift-checker** — Compares artefacts against code; surfaces divergence.
- **bug-fixer / incident-learner** — Bug → reproduction → root cause → fix → sentinel test; feeds findings into the loop.

Open: whether macro and micro reviewer are really separate roles or one role parameterised by tier and aim. Probably the latter, but the per-aim differences in what to look for are large enough that the parameterised form needs to carry them explicitly.

## Ramp-up sequence

How to get from current state to target state without trying everything at once.

### Where I am

I work with models on features. I review. I do parts of this informally. Gaps: standing practices aren't on a cadence; reviews aren't structured around explicit aims; the "make it good" step is ad-hoc; failure modes aren't consistently captured.

### Stage 1 — formalise the per-feature pipeline

For the next feature I touch, run it through the per-feature pipeline above with the aims and phases explicit. Spec gets written down. Reviews are structured against the four aims. Tests are organised by aim. Implementation has an explicit "make it good" step. User guide and demo follow.

Diversity at review time is whatever's available without standing up new infrastructure — at minimum, frontier Claude plus one top-tier OS at the micro tier. Cross-harness if convenient.

The goal of this stage is muscle memory, not perfection. I will do parts of this badly; the next feature will go better.

### Stage 2 — structured review

Macro/micro split explicit. Aims explicit. Cross-harness added where it wasn't already. Diversity stack named and applied deliberately, not opportunistically.

By the end of this stage, I should be able to point at a review and say which tier and which aim it covered, and which tiers and aims are still uncovered.

### Stage 3 — guides and architecture

For the feature I run through Stage 1, write the user guide and demo as part of the same loop. Lift one existing architecture doc toward publication-grade in parallel.

### Stage 4 — standing practices

Add one practice at a time. Suggested order:
1. **Scheduled code reading.** Cheapest, highest insight. Establishes a cadence that the others can hook onto.
2. **Periodic drift checks.** Once the reading habit exists, drift becomes easier to spot; this formalises the spotting.
3. **Accountability checkpoints.** Easy to add once the per-feature pipeline has gates; just name the moments and make the sign-off explicit.
4. **Incident learning loop.** Triggered by incidents, so it adds itself when needed. Sentinel-test protocol should be ready before it's first invoked.
5. **Publication-grade architecture docs.** Quarterly. Slowest cadence; lowest urgency.
6. **Independent re-implementation.** Last and rarest. Only justified once the other practices are running and I can spot the work that warrants it.

### Why this order

Per-feature first because that's where most of the work happens and it produces the biggest immediate payoff. Standing practices second because they amortise over many features and are hard to justify until a per-feature cadence exists. Within standing practices, cheapest and most foundational first.

Stages are not strictly serial. Overlap is fine. A standing practice may emerge naturally before its nominal stage; that's a feature.

## Open questions

- **How explicit do role files at work need to be?** Less than this project, more than nothing. Best resolved by running the per-feature pipeline once and seeing what's hard to keep in my head.
- **Cadence for standing practices.** Numbers above are guesses. Calibrate after starting.
- **Guide-writer as separate role vs folded into spec-helper.** Unresolved.
- **Inherited features.** When I pick up a feature mid-pipeline — spec already written, possibly badly — what's the protocol? Probably: review the existing spec against the aims first; revise before proceeding.
- **Directory state machine.** How much of `proposed/todo/doing/done` to adopt at work. Useful for solo work; less clear when there's a shared codebase with its own conventions.
- **What "publication-grade" means in a work context** — internal-publication, team-readable, org-wide? Probably depends on the doc.

---

Written 2026-05-10 as the prescriptive companion to WORKFLOW.md. Expect revision as the practices meet real features.
