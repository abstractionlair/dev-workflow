# dev-workflow

## Background and Motivation

I've written elsewhere about one aspect of the workflow I've ended up following trying to maximize the value provided by AI --- trying to benefit from a diversity of models and harnesses. This project describes, and implemnts, another aspect. Part of the benefit I think this part brings is independent of the diversity benefit and part of this workflow makes it easier to capture the diversity benefit.

This is a formalish development process which resembles ones in use before we had meaningful AI help. That brings benefits from clear goals, scope discipline, good specs, following good review and test practices, ... all the usual stuff and independent of model and harness diversity. I'd note though that I've never found this kind of formality to deliver anything near what it's promised with human team. There, a little process and formality goes a long way. More was mostly a way to give some people a warm fuzzy feeling that they new what was going on and were in control. Usually false. _However_ it works really well with LLMs. At least today's LLMs. What works best, well, we probably haven't discovered best yet; what works well, keeps changing as models and harneses improve and change over time. And there's no sign of that stopping. I'd guess it's more likely to accelerate.

Where this workflow supports the diversity focused parts of my workflow is in writing all the work out to files, each with a clear purpose and expectations. I can discuss a project's vision with Claude in a long conversation, interacting with the model as, using Anthropic's label, a thought partner, and then have it summarize into a vision document. Then I can have a different model review it, making use of a pre-existing description of what I want out of this kind of document. Then we iterate until its good. That pattern then repeats, step by step, as we work through the stages of getting a project done.

In a sequence of iterations of the general idea, I tried different ways to automate different parts of this but I've steped back from that. Not because they didn't work. Much of it did work or at least showed promise. But I found I was spending the bulk of my time on the methodology and infrastructure than I did on "real" work. Not because it was needed, but because to me this kind of stuff is an attention attractor and sink.

The details below about how it all work are largely written by Claude based on reading the files. It also had access to a lot of logs and "window files" [insert link] which record experience following the workflow. I don't think any of the latter factored into what's below. That's mostly "just the facts". But it could have.

## What this is

A structured AI-augmented development workflow. Importable into any project via Claude Code skills or directly via role files for other harnesses.
Concretely, a distilled process for spec-driven development with multi-model review gates:

```
Vision -> Scope -> Roadmap -> Spec -> Skeleton -> Tests -> Implementation
                                |                           |
                             review                      review
```

Each stage produces a durable artifact. Reviewers gate advancement between stages. Different models can play different roles.

## What's in here

```
ontology.md          Artifact types, relationships, state machine, ownership rules
preamble.md          Shared context loaded by every skill invocation
model-config.json    Role -> preferred model/harness mapping

schemas/             What each artifact must contain
  spec.md            Feature specification schema (the pivotal artifact)
  vision.md          Project vision schema
  scope.md           Project scope schema
  roadmap.md         Feature roadmap schema
  review.md          Review document schema
  bug-report.md      Bug report schema

roles/               Behavioral instructions for each workflow role
  planner.md         Socratic dialogue + writing for vision/scope/roadmap
  spec-writer.md     Socratic dialogue + writing for feature specifications
  macro-reviewer.md  Big-picture review: design quality, spec fidelity (gatekeeper)
  micro-reviewer.md  Line-level code quality review (advisory, no gatekeeping)
  skeleton-writer.md Produces interfaces and type signatures
  test-writer.md     Produces tests (TDD red phase)
  implementer.md     Makes tests pass (TDD green phase)

templates/           Project initialization templates
  workflow-config.json
```

## How to use

### Via Claude Code skills

```
/workflow-init         Set up workflow structure in your project
/workflow-status       See where things stand
/workflow-plan         Strategic planning (vision/scope/roadmap)
/workflow-spec         Write a feature specification
/workflow-review       Review any artifact
/workflow-build        Skeleton -> tests -> implementation
```

### Via other harnesses (Codex, Gemini)

Any harness that can read files works: point the model at the relevant role file. `deploy.yaml` installs this tree to `~/deploy/dev-workflow/` — adjust paths if you keep it elsewhere.

```bash
# Example: GPT-5 reviews a spec
codex "Follow the instructions in ~/deploy/dev-workflow/roles/macro-reviewer.md. Review specs/proposed/my-feature.md against the schema in ~/deploy/dev-workflow/schemas/spec.md."
```

Or use the role launcher, `bin/workflow-role`, which prepares the session context and picks the harness from `model-config.json` (supported harnesses: claude, gemini, codex).

### Progressive formality

Not every project needs the full cascade. `.workflow/config.json` controls which stages are active:

- **Small project**: skip vision/scope/roadmap, go straight to spec -> build
- **Large project**: use all stages for strategic alignment before feature work

## State tracking

Artifacts track state via directory location:

```
specs/proposed/  ->  specs/todo/  ->  specs/doing/  ->  specs/done/
     (draft)         (approved)      (implementing)     (complete)
```

State transitions use `git mv`. Only reviewers (gatekeepers) advance artifacts past quality gates.

## Multi-model coordination

The project directory is the coordination layer. No messaging system needed.

1. Write a spec in Claude terminal
2. Review it in Codex terminal (different model catches different things)
3. Back to Claude for implementation
4. Review implementation in Gemini terminal

Each model reads the current project state from the filesystem. The coordinator routes.

## History

This project supersedes two earlier experiments:

- **[`dev_workflow_meta`](https://github.com/abstractionlair/dev_workflow_meta)** (Feb 2026, reached v1.0) — direct ancestor. Same artifact pipeline (VISION → SCOPE → ROADMAP → SPEC → SKELETON → TEST → IMPLEMENT), same state-via-directory model, same role categories. Current project is a ~3× compression: 22 role files → 7, 11 schemas → 6, helper/writer pairs collapsed, Claude Code skills replace `bin/init-project.sh` + git-submodule bootstrap. Dropped: explicit FeedbackLoops.md / RFC.md process docs, WorkflowExample.md walkthrough, dedicated bug-recorder role.
- **[`MultiModelCLIEmail`](https://github.com/abstractionlair/MultiModelCLIEmail)** (Feb 2026, single-commit prototype) — Maildir-based async messaging between role-mailboxes (coordinator, monitor, explorer-claude, ...) with Mutt as the UI. Current project deliberately *rejects* this mechanism: filesystem-as-coordination instead of message-passing; the coordinator routes synchronously between terminals. What carried over is the underlying observation that roles are not models and the same role can be played by any model.

Neither predecessor is actively maintained. They are read-only references, not dependencies.
