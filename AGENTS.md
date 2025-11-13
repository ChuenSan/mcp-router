# Repository Guidelines

## Project Structure & Module Organization
MCP Router is a PNPM/Turbo monorepo (Node ≥20, pnpm ≥8). Key paths:
- `apps/electron/` hosts the Electron desktop shell plus the Playwright e2e suite under `e2e/`.
- `apps/cli/` exposes the token-based CLI used to connect to Router instances.
- `packages/ui/` provides the shared React + Tailwind component library, while `packages/shared/` and `packages/remote-api-types/` hold cross-app stores and API contracts.
- `packages/tailwind-config/` centralizes design tokens; `public/` stores static assets surfaced in the desktop build; `docs/` and `tools/` keep specs and automation scripts.

## Build, Test, and Development Commands
Run `pnpm install` once per clone, then rely on root scripts:
- `pnpm dev` — launches Turbo dev pipelines for `@mcp_router/shared`, `@mcp_router/ui`, and the Electron app with live reload.
- `pnpm build` — runs `turbo run build` for every workspace; required before packaging or publishing.
- `pnpm make` — triggers Electron Forge packaging (used before release and before e2e).
- `pnpm typecheck` / `pnpm lint:fix` — TypeScript project-wide checks and ESLint + Prettier autofix, mirroring CI.
- `pnpm test:e2e` — packages the desktop app per-arch, copies metadata into `.webpack/<arch>/main`, then executes `apps/electron/e2e` Playwright specs.
- `pnpm knip` — reports unused files/symbols to keep the monorepo lean.

## Coding Style & Naming Conventions
TypeScript + React across packages, using 2-space indentation, semicolons, and named exports where possible. File names stay kebab-case for components (see `packages/ui/src/components/button.tsx`), while hooks/utilities stick to camelCase exports. Reuse the Tailwind tokens from `packages/tailwind-config`, and prefer the shared `cn` helper instead of manual string concatenation. Always run `pnpm lint:fix` (root) or the workspace-level ESLint scripts before opening a PR; the repo-level `eslint.config.mjs` already extends Prettier so no extra formatting pass is needed.

## Testing Guidelines
UI flows are validated via Playwright specs in `apps/electron/e2e/specs/*.spec.ts`. When adding coverage, reuse the existing fixtures (`fixtures/electron-app.ts`, `fixtures/page-objects/*`) and colocate helpers under `e2e/utils`. New specs should describe the scenario in natural language (e.g., `project-switching.spec.ts`) and can run headlessly with `pnpm test:e2e` or interactively with `pnpm --filter @mcp_router/electron test:e2e:headed`. Keep flaky network interactions behind the mocked services already provided in the fixtures and ensure critical user journeys (workspace switching, server toggling, token issuance) stay covered.

## Commit & Pull Request Guidelines
Follow the Conventional Commit prefixes seen in `git log` (`feat:`, `fix:`, `chore:`, `refactor:`). Keep commits scoped to one concern and include workspace hints when helpful (`feat(electron): add workspace switcher`). For pull requests, provide (1) a concise summary plus linked issue/Discord thread, (2) screenshots or short clips for UI-facing changes (desktop app + CLI output), and (3) verification steps covering `pnpm build`, `pnpm typecheck`, and any relevant `pnpm test:e2e` runs. Check that secrets (tokens, API keys) remain in local `.env` files and never enter the repo.

## Security & Configuration Tips
Store local secrets such as `MCPR_TOKEN` or provider keys in `.env.local` files that Electron Forge loads via `dotenv`; never hardcode them. When exercising external MCP servers, point to loopback or mocked endpoints and sanitize logs before sharing. Because Electron packages native modules like `better-sqlite3`, rerun `pnpm make` whenever Node versions or architectures change to rebuild binaries safely.
