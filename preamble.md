# Dev-Workflow Preamble

Load this file at the start of every workflow skill invocation.

## 1. Detect Workflow State

Scan the current project for workflow structure:

```
Read .workflow/config.json if it exists (project settings).
List specs/{proposed,todo,doing,done}/ to find feature specs and their states.
List bugs/{to_fix,fixing,fixed}/ to find bug reports and their states.
List rfcs/open/ to find pending change requests.
Check for docs/VISION.md, docs/SCOPE.md, docs/ROADMAP.md existence.
```

If none of these directories exist, the project hasn't been initialized. Suggest `/workflow-init`.

## 2. Project Configuration

`.workflow/config.json` controls which stages are active:

```json
{
  "active_stages": ["spec", "skeleton", "tests", "implementation"],
  "skip_stages": ["vision", "scope", "roadmap"]
}
```

If `skip_stages` includes a stage, don't prompt for it. Small projects skip strategic planning by default.

## 3. Ontology Quick Reference

**Artifact flow:** Vision -> Scope -> Roadmap -> Spec -> Skeleton -> Tests -> Implementation

**State flow:** `proposed/` -> `todo/` -> `doing/` -> `done/`

**Ownership:** Only reviewers (gatekeepers) advance artifacts past quality gates. Writers cannot self-approve.

**Branching:** Feature branch created when spec moves to `doing/`. Merged when implementation approved.

Full ontology: `~/projects/dev-workflow/ontology.md`

## 4. Status Reporting

When completing a workflow action, report status as one of:

- **DONE** -- action completed successfully, state advanced
- **DONE_WITH_CONCERNS** -- completed but flagging issues for the user
- **BLOCKED** -- cannot proceed; state what's blocking and what was tried
- **NEEDS_CONTEXT** -- missing information; state exactly what's needed

If a workflow action fails 3 times, stop and escalate to the user. Bad work is worse than no work.

## 5. Artifact References

Schemas (what each artifact must contain): `~/projects/dev-workflow/schemas/`
Role definitions (behavioral instructions): `~/projects/dev-workflow/roles/`
Model preferences (which model for which role): `~/projects/dev-workflow/model-config.json`

## 6. Execution Model

Each session is launched by the user for a single task. The user orchestrates the workflow — choosing which role to run, in which harness, on which artifact.

- **Atomic sessions**: Complete the task defined in `.workflow/active-role.md`, update artifacts, report status, and stop.
- **No self-delegation**: Do not fork agents, chain tasks, or launch follow-up sessions automatically unless the user explicitly asks.
- **Artifact-based handoff**: When your task is done, the artifacts you produced or modified are the handoff to the next session. The user decides what happens next.

## 7. Multi-Harness Awareness

This workflow runs across multiple model harnesses (Claude Code, Gemini CLI, OpenCode). Artifacts in the project directory are the coordination layer between sessions. When you create or modify an artifact, it will be picked up by whichever harness operates next.

Write artifacts that are self-contained and readable without conversation context.
