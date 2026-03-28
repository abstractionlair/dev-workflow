# dev-workflow

A structured AI-augmented development workflow. Importable into any project via Claude Code skills or directly via role files for other harnesses.

## What this is

A distilled process for spec-driven development with multi-model review gates:

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
  planning-helper.md Socratic dialogue for vision/scope/roadmap
  planning-writer.md Produces strategic planning documents
  spec-helper.md     Socratic dialogue for feature specifications
  spec-writer.md     Produces feature specifications
  reviewer.md        Reviews any artifact type (parameterized)
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

### Via other harnesses (Codex, Gemini, OpenCode)

Point the model at the relevant role file:

```bash
# Example: GPT-5 reviews a spec
codex "Follow the instructions in ~/projects/dev-workflow/roles/reviewer.md. Review specs/proposed/my-feature.md against the schema in ~/projects/dev-workflow/schemas/spec.md."
```

Or use the role launcher (Phase 3).

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

Each model reads the current project state from the filesystem. Scott routes.
