# AGENTS Guidelines

This document provides standardized rules and commands for agentic coding agents operating in this repository. It covers build, lint, test workflows, and code‑style expectations across languages used in Grafana.

Note: This repo is polyglot (Go back-end, TypeScript front-end, and tooling). Always verify which language/component you are touching and apply the relevant commands below.

## 1) Build, Lint, Test Commands

- Determine the proper toolchain at a glance:
  - TypeScript/JavaScript tooling: node, npm or yarn, possibly pnpm.
  - Go tooling: go, gofmt, golangci-lint if configured.
  - Makefile may exist to orchestrate common tasks.

- Quick environment check:
  - node -v, npm -v or yarn -v
  - go version
  - if a Makefile exists, note it as the primary orchestrator

- Building the project (best-effort generic guidance):
  - If a top-level Makefile exists: run `make build`.
  - If there is a top-level package.json with a `build` script: run `npm run build` or `yarn build` at the repo root.
  - If there are separate workspaces (e.g., frontend/backend): build them individually if needed, e.g.
    - Frontend: `cd frontend && npm run build` or `yarn build`.
    - Backend: `cd backend && go build ./...` (or follow the repo’s Go build script).
  - If no explicit scripts exist, fall back to language-specific build commands:
    - Go: `go build ./...` (from repo root or respective module dir).
    - TS: `npm run build` or `yarn build` inside the relevant package.

- Linting (keep code healthy):
  - Go: run `golangci-lint run` if a config is present.
  - TS/JS: run `npm run lint` or `yarn lint` at the relevant package root. Consider `eslint` with `--ext .ts,.tsx` if needed.
  - Pre-commit style: run language-specific formatters before linting (see section 2).

- Testing (find and run tests, including single test):
  - Go: run `go test ./...` to execute all tests; to run a single test:
    - `go test ./... -run '^TestName$'`
    - For a specific package: `go test ./path/to/pkg -run '^TestName$'`
  - TS/JS: run `npm test` or `yarn test` at the relevant package; to run a single test with Jest:
    - `npm test -- -t 'TestName'` or `yarn test -t 'TestName'`.
  - For integration/CI tests that require extra flags, consult the package’s test script (e.g., `CI=true npm test`).
  - If using a monorepo tool, prefer running tests in the impacted workspace first.

- Quick example sequences:
  - Build frontend only: `cd frontend && npm run build`.
  - Run a single Go test in a package: `go test ./pkg/math -run '^TestAdd$'`.
  - Run a Jest test by name: `npm test -- -t 'renders list'`.

## 2) Code Style Guidelines

The repository blends Go and TypeScript/JavaScript. Follow language-specific sections below, and apply shared cross-language rules where relevant.

- General cross-language:
  - Treat CI as the single source of truth for lint failures; fix locally when possible.
  - Prefer explicit, readable code over cleverness; add comments for non-obvious logic.
  - Avoid committing secrets or credentials; use environment variables or secret managers.
  - Add or update tests for any behavioral changes; ensure coverage remains reasonable.

- Go (backend):
  - Formatting: `gofmt -w` or `go fmt ./...`; run before commit.
  - Imports: standard library first, then third‑party, then project packages; use `goimports` or an editor/IDE plugin if available.
  - Naming: exported names in CamelCase; local vars in short camelCase; package names lowercase.
  - Errors: propagate errors with context; use `fmt.Errorf("...: %w", err)` or `errors.Wrap` equivalents; avoid `panic` for expected errors.
  - Context: pass `context.Context` through APIs; respect cancellation.
  - Tests: use table-driven tests where suitable; keep tests deterministic and fast.
  - Concurrency: guard shared state, avoid data races; prefer channels or sync primitives; use `-race` when running tests.

- TypeScript/JavaScript (frontend):
  - Formatting: use Prettier with a consistent config; run `prettier --write` on staged files if possible.
  - Linting/Imports: enable ESLint with a consistent rule set; use Import Order plugin to group and order imports.
  - Types: avoid `any` when possible; prefer `unknown` for external input; define clear interfaces/types.
  - Naming: camelCase for variables/functions; PascalCase for classes/interfaces; UPPER_SNAKE for constants if used.
  - Modules: prefer explicit exports; avoid circular dependencies.
  - Error handling: do not swallow errors; throw or surface errors with context; use typed errors where possible.
  - Testing: unit tests should be deterministic and fast; use describe/it blocks; use mocks for dependencies.

- Documentation and comments:
  - Public APIs must have JSDoc / Go doc comments explaining intent and usage.
  - Inline comments only where necessary; avoid noisy commentary on obvious code.

- Dependency and security:
  - Pin versions in lockfiles; avoid upgrading transitive dependencies without review.
  - Scan for secret leaks; use tools like `git-crypt` or secret scanners if available.

- Performance and accessibility:
  - Keep hot paths optimized; profile when necessary; write tests that cover performance-sensitive logic.
  - Ensure UI remains accessible; follow semantic HTML and ARIA best practices where applicable.

- CI/CD expectations:
  - PRs should pass linting, type checks, and unit tests at minimum.
  - Add e2e tests for critical flows if available in the repo.

## 3) Cursor Rules and Copilot Rules

- Cursor Rules: none detected in this repo (no .cursor/rules/ or .cursorrules directory found).
- Copilot Rules: no .github/copilot-instructions.md found; if present, follow its guidance verbatim.
- If such files are added later, ensure agents reference them to align with project policy.

## 4) Quick Start Snippet

- Detect language: if Go files exist, prefer `go test`/`golangci-lint`.
- If TS/JS exist, prefer `npm/yarn` scripts at root or package.json.
- Always run formatters first, then lint, then tests.

## 5) Examples

- Run a single Go test:
```
go test ./pkg/net -run '^TestDial' -v
```

- Run a single Jest test:
```
npm test -- -t 'renders the user card' --silent
```

## 6) Maintenance

- Update this AGENTS.md when project tooling changes; announce in PR text.
- Periodically run a full audit of linters/configs to stay aligned with best practices.
