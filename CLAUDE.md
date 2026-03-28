# Dev-Workflow

This project defines a structured development workflow. It is NOT a project to develop — it's a reference used by skills and other harnesses.

## Structure

- `ontology.md` — artifact types, state machine, ownership rules (read this first)
- `preamble.md` — shared context loaded by every skill
- `schemas/` — what each artifact type must contain
- `roles/` — behavioral instructions for each workflow role
- `model-config.json` — role -> preferred model mapping
- `templates/` — project initialization templates

## When editing

- Schemas define required structure. Changes affect what writers produce and what reviewers check.
- Roles define behavior. Changes affect how models operate when playing that role.
- The ontology is the source of truth for terminology. Keep schemas and roles consistent with it.
- The preamble is injected into every skill. Keep it concise.

## Conventions

- Markdown only, no executable code in this project
- Role files use YAML frontmatter for metadata
- Schemas start with `# Schema: [Name]` and a purpose line
- Target sizes: schemas 200-400 lines, roles 150-300 lines
