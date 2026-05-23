<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Repository Guidelines

## Project Structure & Module Organization
- `src/app/` — Next.js App Router routes, layouts, API routes (`api/login`, `api/logout`, `api/brief`, `api/feed`)
- `src/middleware.ts` — edge auth middleware (session verification, redirect-on-expiry)
- `src/components/` — shared React components
- `src/components/providers/` — Context providers + hooks (tracked companies, chat messages, brief, feed filters)
- `src/lib/` — pure-function utilities (session, dispatcher, selectors, feed-filter params, http helpers, notion)
- `src/data/` — seeded JSON fixtures and types
- `tests/api/`, `tests/lib/`, `tests/e2e/` — mirror source paths (e.g. `src/lib/x.ts` → `tests/lib/x.test.ts`)
- `specs/` — feature specs, plans, task lists for non-trivial work
- Root files for project-wide config only

Prefer small, focused modules. Keep functions short and cohesive.

## Build, Test, and Development Commands
- `bun install` — install dependencies (run before any verification command)
- `bun dev` — run dev server (http://localhost:3000)
- `bun run build` — production build
- `bun run lint` — ESLint
- `bun run typecheck` — type-check without emitting (`tsc --noEmit`)
- `bun run test` — run the unit and API contract test suite (Vitest). Note: `bun test` invokes Bun's native runner instead — always use `bun run test`.
- `bun run test:e2e` — Playwright E2E tests (builds + serves on port 3010)
- `bun run verify` — typecheck + lint + test in one command

## Coding Style & Naming Conventions
- 2 spaces for JSON, YAML, Markdown list indentation
- Descriptive file/module names: `chat-dispatcher`, `match-competitor`
- `PascalCase` for React components and types; `camelCase` for functions/variables
- Short functions, cohesive modules, explicit names over abbreviations

## Testing Guidelines
- Add tests alongside new behavior, not after
- Pre-alpha per-feature scope: ~3 tests (2 contract/unit + 1 E2E) — not full coverage
- Name tests after the unit under test: `chat-dispatcher.test.ts`, mirror source paths
- Include a regression test for each bug fix
- `bun run test` is Vitest (unit + API contract); `bun run test:e2e` is Playwright (builds + serves on 3010)
- Run a single Vitest file: `bun --bun x vitest run tests/lib/session.test.ts`
- Run a single Playwright file: `bun run test:e2e tests/e2e/session-expiry.test.ts`

## Commit & Pull Request Guidelines
Use conventional commits: `<type>: <summary>` (e.g. `feat: add chat dispatcher`, `fix: handle empty input`).

Pull requests should include:
- short description of what changed
- why the change was needed
- test evidence (or note if not yet applicable)
- linked issues when applicable
