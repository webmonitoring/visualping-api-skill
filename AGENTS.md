# Agent Instructions

This repository is the **Visualping API Skill** — a Claude skill that teaches Claude how to generate ready-to-run code snippets (curl, Python, JS) for Visualping's monitoring API, or execute calls directly when the environment permits.

## Project Structure

- `SKILL.md` — skill frontmatter, core principles, workflows, and the guidance Claude follows at runtime.
- `reference/api-reference.md` — endpoint-by-endpoint schema and examples. Source of truth Claude reads when constructing requests.
- `README.md` — user-facing install/usage info.

When adding or changing an endpoint, update **both** `SKILL.md` (Common Workflows row + any cross-cutting notes) and `reference/api-reference.md` (endpoint section with schema and example). They are expected to stay in sync.

## Commit Messages and Pull Request Titles

All commit messages and pull request titles MUST adhere to the [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) specification.

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Types

- **feat**: A new skill capability — e.g. a new endpoint added to `SKILL.md` / `reference/`.
- **fix**: A correction to existing guidance, schema, or example that was wrong.
- **docs**: Changes to non-skill files (`README.md`, this `AGENTS.md`, etc.).
- **refactor**: Restructuring skill content without changing behavior or coverage.
- **chore**: Repo maintenance (dotfiles, metadata, tidying).
- **revert**: Reverts a previous commit.

> **Note on `feat` vs `docs`:** this repo's primary artifact *is* documentation (the skill), but from the consuming agent's perspective, new endpoint support is a new capability — use `feat`. Reserve `docs` for changes to non-skill files like `README.md`.

### Breaking Changes

A commit with a `BREAKING CHANGE:` footer or a `!` after the type/scope signals a change that could break existing skill behavior — e.g. removing an endpoint section, renaming a workflow, or changing skill frontmatter in a way that alters trigger conditions.

### Examples

```
feat: add label management and job launch endpoints
fix: correct interval field type in create-job schema
docs: update README with install instructions
refactor: split report-page guidance into its own section
```

## Conventions

- Do not add emojis to files unless explicitly requested.
- Prefer editing existing files over creating new ones; the repo has a deliberately small surface.
- When adding endpoints, cross-reference related sections rather than duplicating content.
- Keep `SKILL.md` tight — verbose schema detail belongs in `reference/api-reference.md`.
