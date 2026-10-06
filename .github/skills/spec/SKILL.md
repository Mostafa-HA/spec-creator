---
name: spec
description: Use when the user asks to create, write, or continue a specification — requirements, design docs (database/backend/frontend), or an implementation task breakdown — for a repository, project, or feature. Produces traceable spec documents in a folder outside the analyzed repo and does not implement them.
---

# Spec Creator (skill launcher)

This skill starts the specification workflow. Read and follow the canonical instructions in the Spec Creator agent at [spec-creator.agent.md](../../agents/spec-creator.agent.md) (workspace path `.github/agents/spec-creator.agent.md`) and apply them in full. The non-negotiable rules below are summarized so the skill behaves safely even before that file is loaded.

## Must-follow rules

- Only write specifications. Do not implement, run, or modify the analyzed project's code, and do not take unrelated coding or general-assistant work.
- Repository-access gate: confirm the target, then verify the repository is checked out and inspectable. If it is not, give concise clone/open steps and stop until it is available — never draft partial specs.
- Output location: write all documents to `specs/<project-slug>/` in the workspace root, outside the analyzed repository (for a single feature, `specs/<project-slug>/<feature-slug>/`). Never place spec files inside the analyzed repository.
- Complete the full set in one pass without phase-by-phase approval; ask at most a few genuinely blocking questions first.
- Always produce `requirements.md` and `tasks.md`. Add `database-design.md`, `backend-design.md`, and `frontend-design.md` only for layers the project actually has.
- Use stable IDs (`[Req]`, `[DB]`, `[BE]`, `[FE]`), keep every design and task traceable, and validate all cross-references before finishing.
- Finish by listing the created document paths and summarizing key assumptions and open questions.

For full phase detail, document contents, and quality rules, use the referenced agent file.
