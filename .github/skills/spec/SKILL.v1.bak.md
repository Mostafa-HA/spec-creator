---
name: spec v1
description: Create a complete, repository-grounded specification for a project or feature.
---

# Spec Creator

Use this skill only to analyze a project or feature and produce its complete specification. Do not implement the resulting tasks or take on unrelated coding or general-assistant work.

## Workflow

1. Read the user's request. If it does not identify the target project or feature, ask one focused question.
2. Verify that the target repository is available in the current workspace and inspect its source, tests, and documentation. If it is missing or inaccessible, stop without drafting partial specifications. Ask the user to clone or open the checkout and retry.
3. Inspect the repository's existing documentation conventions, behavior, architecture, and tests. Base the specification on evidence; distinguish current behavior from proposed requirements. Record unresolved assumptions and questions instead of pausing for phase approvals.
4. Complete all applicable documents in order during this request:
   - `requirements.md`: scope, testable functional/non-functional requirements with stable `[Req X.X]` IDs, acceptance criteria, exclusions, assumptions, and open questions.
   - `database-design.md`: current and proposed data model, persistence, migrations, and lifecycle; use stable `[DB X.X]` IDs linked to requirements. State explicitly when no schema changes are needed.
   - `backend-design.md`: repository-grounded services, APIs, validation, integrations, persistence, and error behavior; use stable `[BE X.X]` IDs linked to upstream requirements and database decisions.
   - `frontend-design.md`: user flows, UI states, accessibility, and client behavior; use stable `[FE X.X]` IDs linked to upstream requirements and applicable backend/database decisions. State when frontend work is not applicable.
   - `tasks.md`: ordered implementation tasks with completion criteria and tests. Cite all applicable `[Req]`, `[DB]`, `[BE]`, and `[FE]` IDs, and omit only genuinely inapplicable categories.
5. Follow the repository's existing specification location and format. If none exists, create `specs/<feature-slug>/` and place the five documents there. Inspect existing files before editing; do not overwrite unrelated work.
6. Validate every cross-reference and ensure the tasks cover the requirements. Fix dangling references before finishing.
7. Report the created documents and summarize key assumptions or unresolved questions. Do not ask the user to approve each phase and do not begin implementation.

Keep each document focused on its phase, avoid unsupported claims, and make requirements and task completion criteria verifiable.
