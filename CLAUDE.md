# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Approach
- Think before acting. Read existing files before writing code.
- Be concise in output but thorough in reasoning.
- Prefer editing over rewriting whole files.
- Do not re-read files you have already read unless the file may have changed.
- Test your code before declaring done.
- No sycophantic openers or closing fluff.
- Keep solutions simple and direct.
- User instructions always override this file.

## Repository Guidelines

### Project Structure

Documentation lives under `docs/`:

- `docs/SPEC.md` — top-level requirements / TRD for the product.
- `docs/TODOS.md` — repo backlog (open design questions, blockers, priority order).
- `docs/adrs/` — Architecture Decision Records.
- `docs/features/` — per-feature folders; each may contain its own `SPEC.md` for feature-scoped requirements.

### Commands

TBD

### Coding Standards

- 2 spaces for JSON, YAML, Markdown list indentation

### Testing

- Add tests alongside new behavior
- Name tests after the unit under test; mirror source paths
- Regression test for each bug fix

### Git Workflow

- All work shall occur in a git worktree, never directly in the main checkout.
- Create a worktree per branch (e.g., `git worktree add ../lamps-<branch> <branch>`); main stays clean.

### Commit & PRs

- Conventional commits: `<type>: <summary>` (e.g., `feat: add API client`)
- Prefer atomic commits: one logical change per commit, independently revertable. Split unrelated changes; don't bundle a refactor with a feature.
- PR body: what changed, why, test evidence, linked issues

## Skill routing

When the user's request matches an available skill, invoke it via the Skill tool. When in doubt, invoke the skill.

Key routing rules:
- Product ideas/brainstorming → invoke /office-hours
- Strategy/scope → invoke /plan-ceo-review
- Architecture → invoke /plan-eng-review
- Design system/plan review → invoke /design-consultation or /plan-design-review
- Full review pipeline → invoke /autoplan
- Bugs/errors → invoke /investigate
- QA/testing site behavior → invoke /qa or /qa-only
- Code review/diff check → invoke /review
- Visual polish → invoke /design-review
- Ship/deploy/PR → invoke /ship or /land-and-deploy
- Save progress → invoke /context-save
- Resume context → invoke /context-restore

## Current State

TBD
