# AGENTS

This repository uses a Laravel backend with Pest/PHPUnit for tests and a TypeScript + React frontend built with Vite. This file collects the commands and code-style rules an agent (or human) should follow when working here.

Location: `AGENTS.md`

## Quick Commands

- Install dependencies (backend + frontend):
    - PHP: `composer install`
    - Node: `npm install`
- Build assets: `npm run build`
- Start frontend dev server: `npm run dev`
- Run Laravel dev with watchers (local helper): `composer run-script dev`

## Lint / Format / Types

- Format frontend code: `npm run format` (Prettier + tailwind plugin)
- Check formatting: `npm run format:check`
- Lint frontend (fix): `npm run lint`
- Lint frontend (check): `npm run lint:check`
- Run TypeScript check (no emit): `npm run types:check` (runs `tsc --noEmit`)
- PHP linter / formatter: `composer lint` (Laravel Pint configured in `composer.json`)
- Full CI-local checks (approx):
    - `npm run lint:check && npm run format:check && npm run types:check && composer run-script test`

## Tests

Backend tests use Pest (wrapper for PHPUnit).

- Run full test suite (same as CI):
    - `composer test` # runs lint:check + `php artisan test`
    - or directly: `./vendor/bin/pest` (used in CI)

- Run a single test file (Pest / PHPUnit):
    - File: `./vendor/bin/pest tests/Feature/MyFeatureTest.php`
    - Specific test method: `./vendor/bin/pest tests/Feature/MyFeatureTest.php::test_method_name`
    - Using php artisan test with filter: `php artisan test --filter="MyFeatureTest::test_method_name"`
    - Using Pest filter (method name only): `./vendor/bin/pest --filter test_method_name`

- Run tests in a specific directory:
    - `./vendor/bin/pest tests/Unit`

## CI Hints

- GitHub Actions: `.github/workflows/tests.yml` uses Node 22, PHP 8.4/8.5, installs node modules, composer deps, copies `.env.example` and runs `./vendor/bin/pest` after `npm run build`.
- Lint workflow: `.github/workflows/lint.yml` runs `composer lint`, `npm run format`, `npm run lint`.

## Code Style Guidelines

Follow existing tooling: ESLint (modern config), Prettier, Tailwind Prettier plugin, TypeScript strictness and Laravel Pint for PHP. Agents should prefer using and respecting these automated rules rather than introducing ad-hoc style choices.

Formatting & tooling

- Use `npm run format` to auto-format frontend code before committing. Prettier config is used; Prettier + `prettier-plugin-tailwindcss` ensures consistent Tailwind class ordering.
- ESLint is configured in `eslint.config.js`. It includes React (jsx-runtime), stylistic rules for padding lines, and `import/order` enforcement (alphabetized groups).
- PHP formatting and linting: use `composer lint` (Pint). The repository's composer scripts and GitHub actions expect Pint to be used in CI.

TypeScript / JavaScript rules

- tsconfig.json enforces `strict` and `noImplicitAny`. Treat types as first-class. Use `--noEmit` type checking via `npm run types:check` when validating builds.
- Prefer `type` imports (enforced): use `import type { Foo } from './types'` when importing only types. ESLint is set to prefer `type-imports` with `separate-type-imports` fix style.
- Avoid mixing type-only imports with runtime imports on the same line — keep them separate to match `consistent-type-imports` rules.
- The project allows `any` in practice (eslint `@typescript-eslint/no-explicit-any` is off), but prefer precise types or `unknown` for catches. When `any` is used, leave a short justification comment.
- `jsx` runtime is `react-jsx`. For components prefer explicit props interfaces and annotate return types when helpful.

Imports

- Order imports using the `import/order` grouping: builtin, external, internal, parent, sibling, index. Alphabetize within groups (case-insensitive).
- Use the path alias defined in `tsconfig.json`: `@/*` -> `resources/js/*`. Prefer alias for internal frontend modules.
- Use named imports over default imports unless the package exports a natural default. The ESLint config does not force this strongly but consistent named imports help readability.

React / UI rules

- Components: PascalCase for component names and filenames (e.g., `UserCard.tsx`, `UserCard` component).
- Small presentational components: keep them focused, prefer props over implicit context.
- Prefer function components (no `React.FC` requirement). Type props explicitly:
    - `interface Props { ... }
export default function MyComponent({ foo }: Props) { ... }`
- Hooks: follow rules of hooks; ESLint plugin for react-hooks is enabled. Keep hook usage in top-level function scope.

Naming conventions

- Variables & functions: camelCase (e.g., `fetchUser`, `userId`).
- React components, classes, types, and interfaces: PascalCase (e.g., `UserCard`, `UserProfileProps`).
- Types & interfaces: Use suffix `Props` for component props, `DTO` / `Input` when appropriate.
- Constants: UPPER_SNAKE_CASE for configuration constants exported from modules (e.g., `API_TIMEOUT_MS`).

Error handling

- PHP / Laravel:
    - Use FormRequest validation and return structured validation responses when applicable.
    - Throw exceptions for unexpected states. Catch and handle exceptions at boundaries (controllers, jobs, console commands) and log appropriately via `Log::error()`.
    - Do not silently swallow exceptions — always log or rethrow unless intentionally ignored (document with a comment).
- JavaScript / TypeScript:
    - Prefer returning `Promise` rejections for async failures and handle them at the caller or boundary (UI error states).
    - In `catch` blocks, prefer typed handling: if you need to inspect an error, treat it as `unknown` and narrow it before accessing properties.
    - Log unexpected errors to console only in local/dev; production logging should use server-side logging or analytics.

Testing practices

- Keep tests fast and focused. Use factories (database/factories) for Laravel tests.
- For a failing test, run the single-file / single-method commands above to iterate quickly.
- When adding tests, update `composer.json` scripts if needed and make sure `composer test` still runs successfully.

Git / Commit / PR

- Keep commits small and focused. Run `npm run format` and `npm run lint` before committing.
- PRs should pass the GitHub Actions: lint + format + tests. If a formatting change is required, prefer to include it in a separate commit labeled `chore: format`.

Cursor / Copilot rules

- Cursor rules: none found in this repository (`.cursor/rules/` or `.cursorrules` not present).
- GitHub Copilot instructions: none found at `.github/copilot-instructions.md`.

If you find new repository-level rules (Cursor, Copilot, or other IDE policies), add them to this file and configure CI to enforce them.

Where to look

- Frontend configuration: `package.json`, `tsconfig.json`, `eslint.config.js`.
- PHP/Laravel configuration & scripts: `composer.json`, `phpunit.xml` (if present), `.github/workflows/*.yml`.

Next steps for agents

1. Run `npm ci && composer install` then `npm run types:check` to verify environment.
2. Run `npm run lint` and `composer lint` and fix any linter issues before making changes.
3. When editing tests, run a single test file using `./vendor/bin/pest path/to/Test.php` and iterate.

If you want, I can also:

1. Add repository git hooks (pre-commit) to run format + lint (ask explicitly).
2. Create a CONTRIBUTING.md with developer onboarding steps.
