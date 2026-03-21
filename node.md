# Node.js

## Quick Reference

| Tool                       | AI-Friendly Flags                                                       |
|----------------------------|-------------------------------------------------------------------------|
| npm install/ci             | `--no-progress --no-fund --no-audit`                                    |
| npm run                    | No extra flags needed                                                   |
| npx                        | `--yes` (skip install prompts)                                          |
| yarn install               | `--no-progress --non-interactive`                                       |
| pnpm install               | `--reporter=silent` (or `--reporter=append-only`)                       |
| eslint                     | `--no-color` (default output is already clean)                          |
| prettier                   | `-l --no-color` (only list files needing formatting, no summary noise)  |
| biome check                | `--reporter=summary --colors=off`                                       |
| jest                       | `--no-color --verbose`                                                  |
| vitest                     | `vitest run --reporter=agent --no-color` (hides passing tests)          |
| mocha                      | `--reporter=min --no-color`                                             |
| tsc                        | `--noEmit --pretty false`                                               |
| playwright test            | `--reporter=line`                                                       |
| cypress run                | `-q --reporter=min`                                                     |

## npm

```bash
# Install — minimal output
npm install --no-progress --no-fund --no-audit

# Quiet install (warnings and errors only)
npm install -q

# Run scripts
npm run build
```

- `--no-progress` — disables download progress bar
- `--no-fund` — suppresses "X packages are looking for funding" noise
- `--no-audit` — skips vulnerability audit on install (saves network call + output)
- `-q` / `--quiet` — only shows warnings and errors

Env var equivalents: `npm_config_progress=false`, `npm_config_fund=false`, `npm_config_audit=false`.

`npm ci` accepts the same flags and is preferred in CI — it does a clean install from lockfile.

## npx

```bash
npx --yes some-package
```

- `--yes` / `-y` — auto-confirms install prompts (prevents blocking on "Ok to proceed?")

Without `--yes`, npx prompts interactively when a package needs to be installed, which hangs AI agents.

## yarn (v1 Classic)

```bash
yarn install --no-progress --non-interactive
```

- `--no-progress` — disables download progress bar
- `--non-interactive` — disables interactive prompts
- `--json` — outputs structured JSON logs (one JSON object per line)
- `--no-emoji` — disables emoji in output

Env var: `CI=true` auto-enables `--non-interactive` and `--no-progress`.

For Yarn Berry (v2+), flags differ. Berry uses `yarn install --immutable` for CI and has different reporter options. See [Yarn Berry CLI docs](https://yarnpkg.com/cli/install).

## pnpm

```bash
# Silent install
pnpm install --reporter=silent

# Minimal output (appends instead of rewriting lines)
pnpm install --reporter=append-only
```

- `--reporter=silent` — suppresses all output, including fatal errors
- `--reporter=append-only` — removes progress bar animations, appends lines instead (good for non-TTY)

pnpm auto-detects non-TTY environments and defaults to `append-only`. Use `--reporter=silent` for zero output — but note it suppresses even fatal errors, so `append-only` is safer for most agent workflows.

## eslint

```bash
npx eslint src/ --no-color
```

- `--no-color` — disables ANSI colors (also respects `NO_COLOR=1` env var)

ESLint v9+ default output is already compact — one error per line with rule names.

- Default format shows: `file:line:col  severity  message  rule-name`
- `--format json` — structured JSON output (very verbose, includes full source)

ESLint v9 removed built-in `compact` and `unix` formatters. Install separately if needed: `npm install -D eslint-formatter-compact`.

For auto-fix: `npx eslint src/ --fix --no-color` — ESLint is a good fixer, let it apply changes directly.

## prettier

```bash
# Check which files need formatting (most token-efficient)
npx prettier -l src/

# Fix formatting
npx prettier --write src/
```

- `-l` / `--list-different` — prints only file paths that need formatting, one per line
- `--check` / `-c` — lists files and appends a human-friendly summary ("Code style issues found...") — noisier than `-l`
- `--no-color` — disables colors
- `--no-error-on-unmatched-pattern` — prevents errors when globs match nothing
- `--log-level silent` — suppresses all non-error output

Prettier is a fixer — use `--write` to fix directly. Use `-l` to check.

## biome

```bash
# Compact summary (lint + format)
npx biome check src/ --reporter=summary --colors=off

# Fix
npx biome check src/ --write --colors=off
```

- `--reporter=summary` — compact output: lists files with violation counts and rules, no code snippets
- `--reporter=github` — one-line-per-error format (GitHub Actions annotations)
- `--colors=off` — disables colors
- `--write` — applies safe fixes

Default output is very verbose — includes full code snippets and diffs. The `summary` reporter is dramatically more token-efficient.

Other reporters: `json`, `json-pretty`, `junit`, `github`, `gitlab`, `summary`, `checkstyle`, `rdjson`, `sarif`.

## jest

```bash
npx jest --no-color --verbose
```

- `--no-color` — disables ANSI colors. Env var alternative: `FORCE_COLOR=0`
- `--verbose` — shows individual test names (useful for agents to understand what passed/failed)
- `--silent` — suppresses `console.log` from tests (still shows failures)
- `--json` — outputs structured JSON (very verbose, includes all test details)
- `--ci` — prevents snapshot auto-updates (recommended for agent runs)

Single test: `npx jest --no-color path/to/test.test.js` or `npx jest --no-color -t "test name"`

The banner (version, time, suites summary) is always printed. The main savings come from `--no-color` — Jest's colored output includes many ANSI escape sequences.

## vitest

```bash
npx vitest run --reporter=agent --no-color
```

- `--reporter=agent` — **best for AI agents**: hides passing tests, only shows failures. Added in Vitest 4.1+ and auto-activates when it detects an AI agent environment
- `--reporter=dot` — fallback for older Vitest versions: minimal dots for pass/fail, details only on failure
- `--reporter=default` — shows all test names with pass/fail status
- `--no-color` — disables colors
- `--silent` — suppresses `console.log` from tests
- `run` — runs once without watch mode (critical: without `run`, vitest enters watch mode)

Single test: `npx vitest run path/to/test.test.ts` or `npx vitest run -t "test name"`

Other reporters: `verbose`, `json`, `junit`, `tap`, `tap-flat`, `github-actions`, `tree`, `html`, `blob`, `hanging-process`.

## mocha

```bash
npx mocha --reporter=min --no-color
```

- `--reporter=min` — minimal: only shows summary count and failure details
- `--reporter=dot` — dots for progress, failure details below
- `--reporter=json` — structured JSON output
- `--no-color` — disables colors
- `--exit` — force exit after tests (prevents hanging on open handles)
- `--dry-run` — reports tests without executing test bodies (hooks still run). Useful for verifying test discovery

The default `spec` reporter lists every test name with indentation — verbose but readable. `min` is the most token-efficient.

Other reporters: `spec` (default), `list`, `progress`, `json-stream`, `xunit`.

## tsc (TypeScript)

```bash
npx tsc --noEmit --pretty false
```

- `--pretty false` — one error per line: `file(line,col): error TSxxxx: message`
- `--noEmit` — type-check only, don't produce output files
- Default (or `--pretty true`) adds multi-line errors with ANSI colors and code snippets

In non-TTY environments, tsc defaults to `--pretty false` automatically. But it's safer to be explicit.

Tip: Use `--skipLibCheck` to skip type-checking all `.d.ts` declaration files (including those in `node_modules`) — avoids noise from third-party type conflicts but also suppresses errors in your own declaration files.

## playwright test

```bash
npx playwright test --reporter=line
```

- `--reporter=line` — one line per test with file:line on failure
- `--reporter=dot` — minimal dots
- `--reporter=list` — default, lists all tests with status
- `--reporter=json` — structured JSON
- `--reporter=github` — GitHub Actions annotations
- `--quiet` — suppresses stdio from test processes (reporter output still prints)

Playwright's `line` reporter is the best balance of info and token efficiency.

Other useful flags: `--workers=1` (serial execution), `--max-failures=5` (stop after N failures).

## cypress run

```bash
npx cypress run -q --reporter=min
```

- `-q` / `--quiet` — suppresses Cypress banner and most output, only shows reporter output
- `--reporter=min` — minimal mocha output (Cypress uses mocha reporters)
- `--reporter=json` — structured JSON output

Without `-q`, Cypress prints a large banner with version, browser info, and spec file headers.

## AGENTS.md / CLAUDE.md Template

Copy this into your project's AGENTS.md or CLAUDE.md:

```markdown
## Running Tools

- Install: `npm install --no-progress --no-fund --no-audit`
- Tests (Jest): `npx jest --no-color --verbose`
- Tests (single): `npx jest --no-color path/to/test.test.js`
- Tests (Vitest): `npx vitest run --reporter=agent --no-color`
- Type check: `npx tsc --noEmit --pretty false`
- Lint: `npx eslint src/ --no-color`
- Lint fix: `npx eslint src/ --fix --no-color`
- Format check: `npx prettier -l --no-color src/`
- Format fix: `npx prettier --write --no-color src/`
```
