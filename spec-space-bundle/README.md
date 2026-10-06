# Spec Creator — Copilot Space setup

This bundle lets any developer use the Spec Creator workflow from a shared **Copilot Space** — no local install required — and optionally pull the files for local `/spec` use.

## What's in this bundle

| File | Purpose |
| --- | --- |
| `spec-space-instructions.md` | Self-contained instructions to paste into the Space's **Instructions** field. This is what makes the Space behave as the Spec Creator. |
| `spec-creator.agent.md` | The VS Code custom agent file, for users who also want it locally. |
| `SKILL.md` | The VS Code Agent Skill (`/spec`), for users who also want it locally. |
| `README.md` | This file. |

> The local files (`spec-creator.agent.md`, `SKILL.md`) are copies kept here so a Space user can download the whole bundle in one place. The authoritative local versions live in the source repo under `.github/agents/` and `.github/skills/spec/`.

## One-time setup (owner)

1. Go to **github.com/copilot/spaces** and create a Space (for example, "Spec Creator").
2. Open the Space's **Instructions** and paste the full contents of `spec-space-instructions.md`.
3. **Attach sources** so Copilot has something to analyze and users have something to pull:
   - Attach this bundle (or the source repo) so `spec-creator.agent.md` and `SKILL.md` are downloadable from the Space.
   - For each analysis, attach (or have the user attach) the target repository to the Space.
4. **Share** the Space with your org or the intended users (Space sharing settings).

## How any user uses it (no install)

1. Open the shared Space.
2. Attach or select the repository to specify (or paste the relevant files).
3. Ask, e.g.: "Create a spec for this repository" or "Spec the checkout feature."
4. Copilot follows the Space instructions and returns `requirements.md`, `tasks.md`, and the applicable design docs. Save them under `specs/<project-slug>/`, outside the analyzed repo.

## How a user pulls it for local `/spec` (optional)

1. From the Space, download `spec-creator.agent.md` and `SKILL.md`.
2. Place them in their workspace:
   - `.github/agents/spec-creator.agent.md`
   - `.github/skills/spec/SKILL.md`
3. Reload VS Code. They can now trigger the skill with `/spec`.

> Note: the local `SKILL.md` references the agent by a relative path (`../../agents/...`). Keep both files in the `.github` layout above, or use the self-contained `spec-space-instructions.md` content if you want a single standalone file.

## Why a Space (and its limits)

- ✅ Central, shareable, versionable; zero local install to start using it.
- ✅ Grounds Copilot in an attached repo for evidence-based specs.
- ⚠️ A Space does not register a `/spec` slash command in anyone's editor — that requires the local files above or a packaged extension.
- ⚠️ A Space chat may not write files to disk; it returns documents to copy and save.
