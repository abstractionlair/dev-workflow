# Workflow

Notes for me on what I'm doing, why I trust it, and what I'm still working out. Also for fresh AI sessions: the framing section below captures the argument I've already had with myself, so settled questions don't get reopened. Calibrated to the public capabilities of frontier models in 2026; revisit as that changes.

## Why this shape

The workflow's purpose is producing good software. My accountability for what ships shapes how it has to deliver that — I can't safely step back from details unless there are structural reasons to trust the work was done well.

The beliefs about humans and current AI that materially shape this workflow:

- AI is more reliable working from explicit specifications than from implicit goals. The better the spec, the more reliable the implementation.
- AI is very effective at applying clear standards across large quantities of code, beyond what a linter can enforce.
- Multi-model review delivers partial independence. Useful, but not as strong as "different priors and different blind spots" implies — models share training data and have correlated failure modes.
- Humans have implicit context and integrate distantly-related experience in ways current AI does not. Human judgment remains the routing layer.
- I'm accountable for what ships. The AI is not, and cannot be — there is no functional way to make an AI bear accountability the way a human does. The workflow has to deliver good software given that constraint.

Other things are true about the human/AI asymmetry — speed, long-form continuity, willingness to do overhead work — but they aren't what's driving the shape of the workflow.

Some forms of formal process are bad-fit for human developers. Writing a behavioral contract before code, having a different person review every interface change, producing a structured review document instead of a verbal sign-off — these costs land on humans whose comparative advantage is implicit reasoning, and who reasonably skip the overhead. The same formality is well-suited to current AI, whose comparative advantage is following explicit process. The workflow gives AI what it needs to be effective; the human collects the byproduct of work that is both faster and better-documented than they could have produced alone.

### Principles

- **Specifications, decision records, vision docs, and architecture documentation carry intent across time and participants — and serve as long-term memory for models.** They ground subsequent work and give a fresh model session what it needs to act usefully without inheriting the original conversation. The two functions are orthogonal — one is about communication across roles, the other about extending model context past the conversational horizon. (Code is also an artifact, but doesn't serve these functions in the same way.)
- **Each artifact is a contract for the next stage.** A spec must be precise enough that downstream work can proceed without going back to the human. An artifact too vague to function as a contract is not done.
- **Reviews and tests verify we built what we set out to build, and that we built it correctly.** This is their purpose. Other framings (gate-keeping, quality control) are downstream of this.
- **Diversity of perspective is one of several trust mechanisms.** A second reviewer from a different model catches what the writer missed — though independence is partial. Tests are also a trust mechanism, as are types and contracts, lint and static analysis, and the human's own attention. Diversity should not be treated as if it were the only mechanism in play.
- **Engagement scales with abstraction.** I collaborate closely with AI on the high-level work — vision, scope, architecture — and engage more lightly with implementation details. The model still writes, still proposes, still reviews at every level; the question is how much of my attention each level deserves.
- **The human is the router and the decider.** Models propose, models review, models implement. I decide which proposals to accept, which findings to act on, when to override consensus, and when to stop.

## What I'm doing

I've stepped back from line-by-line implementation. Most of my time goes into specifications, tests, architecture decisions, user guides and demos for new components, and reading what models tell me about the code. I read implementation code for structure and smell, more carefully when something fails.

This is good for the work and good for me. I am faster than I was, and what gets produced is more legible — better-specified, better-tested, better-documented — than what I produced alone. The shift is not a retreat. My comparative advantage is implicit knowledge and a wide field of view; the agents' is taking explicit goals and making them happen. Letting each do what it does well is the point.

What this captures is the phase before the formal SDLC begins. I'm the only human involved at this stage, but the work itself is already collaborative with multiple models. At work, what I produce here eventually goes into review by colleagues, automated testing and lint, AI review by other teams using prompts I haven't written.

## What I'm worried about

Three things, in increasing order of how much they actually keep me up.

**I am delivering things I don't fully understand.** When I haven't written the code, my mental model of the system is whatever I've reconstructed from specs, tests, user guides, and review reports. That is a real model — but it's a sketch, not a map. There are details I never see, decisions that never become explicit, and conventions that emerge in the code without my noticing.

**Things that work today may not work later.** Production code lives. Dependencies change. New code gets layered on. Things that pass tests today may quietly stop matching their specs as the surroundings shift. If I haven't internalized the original, I won't see the drift.

**One day something will break and I may not know how to fix it.** This is the one. The system is mine to keep running. If a model wrote a subsystem six months ago and that subsystem starts failing, I am the person who has to figure out what went wrong. If my mental model is too thin, I am in trouble — and so is whoever depends on the system.

These are risks the workflow has to address.

## What I do, and why I trust it

Most of these practices serve more than one purpose: they are what current AI needs to be effective, what gives current AI long-term memory across sessions, and what gives me a way back into the system later — including with fresh AI sessions I bring in to help me understand code I no longer hold in my head.

### Specs are written collaboratively

Specs are not handed from me to an agent as instructions. They're written together, in a sustained bidirectional conversation — I bring intent and context; the agent surfaces ambiguity, asks questions, proposes precision; I read, react in detail, push back, redirect; we iterate over many turns, sometimes many sessions. The spec is finished when reading it tells me what I actually meant, and when I'm confident a downstream consumer (test writer, implementer, future model session) can proceed without going back to me.

This is also where the AI's leverage for me is highest. A good spec lets the rest of the work happen with much less of my attention. Time spent here is not overhead — it's the thing that makes the rest of the workflow safe.

### User guides and demos

Most of my work involves adding new classes, components, or subsystems. Alongside the code, I work with a model on a user's guide and demos — how the new thing is used, examples that show it in action, the abstractions made concrete.

These are paired deliverables. The user's guide is the prose; the demos are the executable counterpart. Both are written collaboratively, with the same close back-and-forth as a spec — multi-point comments on drafts, iteration on wording and clarity for the guide, iteration on what each demo should illustrate and how. As intensive a writing engagement as I have anywhere in the workflow.

I run the demos. They're scaled to whatever I've added or changed, so running them shows me what I built doing what it's supposed to do.

These guides and demos serve several audiences: me, returning to the component months later; fresh model sessions that need to use it without having seen it built; and colleagues in several roles — as reviewers of the code, as users of the new functionality, and as future developers who'll extend or modify it.

### Tests are read at the right level

Tests are the most direct expression of what was actually built. If I want to know what the system promises, I read the tests, not the implementation. But "read carefully" doesn't mean line-by-line. I read tests test-by-test, checking that the things I want tested are tested, and that the ways things might go wrong are tested too. When the tests don't match what they're verifying, the right move is to fix the tests before doing anything else.

Multi-model test writing helps here. Different models test for different things, and the union of two independently-produced suites is a stronger description of the system's promises than either alone. Independence is partial — models share training data and have correlated failure modes — but the gain is real.

### Architecture documentation

These docs serve me re-entering the work after time away, fresh model sessions that need the architecture without reconstructing it from code, and — when they meet the bar — colleagues and future developers working with or extending the system. Not all of mine currently meet that bar. Some are notes-quality: good enough for me and a returning model session, but needing extra diligence before I'd point others to them. Producing publication-grade versions is something I want to make part of the routine. All audiences depend on the docs being current.

I keep an architecture-level description of each system: a system map showing the major pieces and how they fit, plus a guidelines doc capturing the conventions that have emerged. They're produced as the system grows and updated when it changes.

How current they actually stay is a live question. "Living document" is easy to say; updating on every change is not realistic, and some delay is fine. I have not yet found the right rule for how much delay is too much. This is one of the practices I'm still calibrating, not one I have nailed.

When the docs are current, they're a backstop against my mental model going stale, a way for a fresh model session to get oriented, and a starting point for colleagues. When they aren't, the system gets harder to come back to for any of us.

### Reviews — purpose, structure, diversity

Reviews exist to confirm we built what we set out to build, and built it correctly. They're also one of the channels through which I learn what the system actually contains — a review that surfaces something surprising about my own spec is doing exactly what I need it to do.

**Aims.** A good review answers four distinct questions:

1. *Did we build what we set out to build?* End-to-end correctness against the spec.
2. *What could go wrong?* Failure modes — what happens when inputs are bad, dependencies fail, assumptions are violated.
3. *Are there line-level bugs?* Off-by-one errors, edge cases, type mismatches.
4. *Is the code well-made?* Concise, idiomatic, non-redundant, with capabilities divided at sensible joints.

Tests should cover the same dimensions, organized to reflect them rather than as a flat list.

**Macro and micro.** Reviews split into macro (structural, against schema and upstream artifacts) and micro (line-level). The split maps to model capability tiers: macro work needs frontier models; micro work can be done well by top-tier open-source models. The dimensions cross-cut the aims — most aim 1 and aim-4-architectural questions are macro; aim 3 and aim-4-local are micro; aim 2 spans both.

**Asking reviewers.** Review prompts should ask specific questions checked against external standards, not request adversarial criticism. "Be skeptical and find what's wrong" gets parsed as "the user wants me to dislike this," and the model produces criticism whether real problems exist or not. False-positive reviews erode trust in review signal and are worse than rubber-stamp reviews.

Better prompts:
- "Review this spec against the schema. For each schema requirement, note whether it's satisfied."
- "Review this code against the spec. List specific inputs or conditions where it would not match what the spec requires."
- "The spec author documented these failure modes [list]. Review whether the implementation addresses each."

The structure: anchor against something concrete (schema, spec, documented failure modes) and ask the reviewer to map the artifact to the anchor. "Did this satisfy criterion X?" is testable; "find what's wrong" is open-ended and prompts confabulation.

**Independent perspectives come from several axes.** None is the diversity mechanism; they compound.

- *Different vendors.* Anthropic, OpenAI, Google for proprietary frontier models; top-tier open-source models from yet other lineages. Different training pipelines, different priors. The strongest diversity available.
- *Different model tiers within a vendor.* Opus/Sonnet/Haiku from Anthropic, for instance — different capacities and characteristic failure modes from related training.
- *Different harnesses.* Same model in Claude Code, OpenCode, or an internal harness — different system prompts, tools, agent definitions, conversation context. Different reasoning trajectories from the same priors.
- *Different prompts within a harness.* Same model with skeptical, naive-user, or security-focused framings.

Independence is partial across all of these — models share training data and have correlated failure modes. The gain from a second reviewer is real, but smaller than fully independent review would deliver.

**At work.** I don't have multiple proprietary frontier vendors there — Claude is the only one available to me. The rest of the diversity stack still works: open-source plus cross-tier plus cross-harness plus cross-prompt covers more ground than I'd assumed, especially with stronger compensations elsewhere:

- Front-load risk into spec and failure-mode documentation. If review is weaker, the spec has to be stronger.
- Tests carry more of the verification load.
- I pay more attention at the architecture level — providing the perspective the second proprietary frontier model can't.

**Reading reviews.** I read reviews. Even short ones. If I'm not learning anything from a review, either the review is wrong, the artifact was too vague to be reviewable, or I have stopped paying attention — and any of those is worth catching.

### Debugging

When something breaks — error, stack trace, unexpected behavior — I usually start by asking a model to find the cause. The model is usually right. I verify by looking at the code it points to.

Finding the cause is frequently far harder than verifying it. That asymmetry is one of the larger wins of this division of labor when it holds: I get the diagnosis with a fraction of the effort solo would have taken. It doesn't always hold — race conditions, intermittent failures, and bugs that depend on state can be hard to find *and* hard to verify. But the frequent-asymmetry case covers a lot of ground.

The flip side: when the model can't identify the cause, I dig in. Still in collaboration — I pair-read code with the model, propose hypotheses, ask it to falsify them, run experiments. Cases where the model gets stuck are fertile ground for developing the mental model I'd been deferring.

Beyond debugging, I aim to read code regularly to keep some sense of the texture of the system. In practice that happens less than I'd like — most routine code reading happens through the demos. When something fails, I read the relevant subsystem then.

Development-time failures are a real learning channel: the diagnosis-and-verification pattern surfaces parts of the code I had not seen, and the error itself often points at architectural or code issues worth improving rather than just patching past.

### I dial it back when stakes are low

The workflow as described is calibrated to serious work — production systems, things other people depend on, things where the cost of getting it wrong matters. A throwaway prototype, a one-off script, an internal tool with three users gets less. The discipline is consistency: whichever stages I use, I produce them and review them in order.

The risk of dialing back is that I get used to dialing back, and the version of me three months from now does it for things that should have had the full treatment. I notice this and resist it.

## And it's working

The shift is good for me. The output is more legible and faster than what I produced alone, the worries above each have a structural response, and the practices suit my comparative advantage rather than fight it. The questions in the next section are real, but they are gardening — places where the practice is still maturing — not foundation issues.

## Things I am still working out

Existing practices being calibrated:

- **How current the architecture documentation actually has to be.** Some delay is fine; I do not yet have a good rule for how much.
- **When to accept a review I disagree with.** Reviewer findings I think are wrong are sometimes right. Reviewer findings I think are right are sometimes worth overriding. I rely on judgment here and have not articulated it to myself.
- **How to keep the reading habit when I'm busy.** Routine code reading is the first thing to drop when I'm pressed for time, and one of the more important things to keep.
- **How to keep test quality from drifting.** Tests written six months ago may not match how I'd write them now, and may not match how the spec evolved. There has to be some routine for revisiting them. I haven't built it.

Additions I want to make:

- **Failure-mode documentation alongside features.** At spec time, write down how the feature is expected to fail — which inputs break it, which dependencies are fragile, which assumptions might not hold. Gives reviewers and testers concrete things to check against rather than an open invitation to invent problems.
- **Periodic drift checks.** A routine where I (with AI help) periodically re-derive the architecture documentation from current code, or run comprehensive probes against the live system. Closes the loop on "things that work today may not work later."
- **Active scheduled code reading.** Short windows where a model walks me through a piece of code asking me what each part does and what would happen if X. Tests my mental model rather than just refreshing it.
- **Explicit accountability checkpoints.** A few sentences from me at merge on what I'm vouching for. Not "the tests pass" — what specifically I've verified, what I've delegated to the reviews, what assumptions I'm betting are right.
- **Incident learning loop.** When something breaks, a routine asks: what review or test could have caught this? Failure data is the highest-quality input for tuning testing and review priorities.
- **Independent re-implementation for the highest-stakes pieces.** Two models implement the same spec; I compare their outputs on a battery of test inputs. Where they disagree is where errors hide. Heavier than review; reserve for things where the cost of being wrong is large.
- **Architecture documentation worth publishing.** Some of mine are notes-quality — good enough for me and a returning model session, but not for colleagues. Producing publication-grade versions, with the diligence to back that up, should be part of the routine.

---

*Written 2026-05-10. Revised same day after detailed pass. This is in motion; expect revision.*
