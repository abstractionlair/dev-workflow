---
name: workflow-plan
description: Strategic planning — vision, scope, and roadmap via Socratic dialogue then writing. Use when starting a large project that needs strategic alignment before feature work.
---

# Strategic Planning

Define the project's vision, scope, and roadmap. Two modes: **explore** (Socratic dialogue) and **write** (produce the document).

## Process

### 1. Load context

```
Read ~/deploy/dev-workflow/preamble.md
```

Follow the preamble instructions. Check `.workflow/config.json` — if strategic planning stages are in `skip_stages`, tell the user: "Strategic planning is skipped for this project. Update `.workflow/config.json` to enable, or jump to `/workflow-spec`."

### 2. Determine which artifact

If the user specified (e.g., "write the vision"), use that.

Otherwise, follow the cascade:
1. If no `docs/VISION.md` exists (or only stub): start with Vision
2. If vision exists but no `docs/SCOPE.md`: start with Scope
3. If both exist but no `docs/ROADMAP.md`: start with Roadmap
4. If all exist: "Strategic planning complete. Next: `/workflow-spec` for feature work."

### 3. Determine mode

If the user wants to think through the artifact, go to **Explore** mode.
If the user is ready to write, go to **Write** mode.
Default: start with Explore for the first artifact, Write for subsequent ones.

### 4. Explore mode (planning-helper)

```
Read ~/deploy/dev-workflow/roles/planning-helper.md
```

Follow the planning-helper role instructions, parameterized by artifact type:

**Vision exploration**: Why does this project exist? What problem does it solve? Who benefits? What does success look like in 2-5 years?

**Scope exploration**: What's in scope? What's explicitly out? What are the constraints? What are the dependencies? What's deferred to later?

**Roadmap exploration**: What features are needed? In what order? What are the dependencies between features? What's the minimum viable first phase?

When thinking is clear, transition to Write mode.

### 5. Write mode (planning-writer)

```
Read ~/deploy/dev-workflow/roles/planning-writer.md
Read ~/deploy/dev-workflow/schemas/<type>.md
```

Where `<type>` is `vision`, `scope`, or `roadmap`.

Follow the planning-writer role. Write the artifact to the appropriate location:
- `docs/VISION.md`
- `docs/SCOPE.md`
- `docs/ROADMAP.md`

### 6. After writing

Suggest review if desired: "Consider reviewing this with a different model for a fresh perspective."

Then suggest the next step in the cascade:
- After Vision: "Next: define scope with `/workflow-plan`"
- After Scope: "Next: plan the roadmap with `/workflow-plan`"
- After Roadmap: "Strategic planning complete. Start feature work with `/workflow-spec`"

Report status: **DONE** with path to the created artifact.
