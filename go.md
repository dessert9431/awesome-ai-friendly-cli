# Go

## Quick Reference

| Tool              | AI-Friendly Flags                                                         |
|-------------------|---------------------------------------------------------------------------|
| go build          | No flags needed (no colors, no progress bars)                             |
| go test           | `-v` (individual test names), `-json` (NDJSON)                            |
| go vet            | No flags needed (output is `file:line:col: message`)                      |
| go mod tidy       | No flags needed (silent on success)                                       |
| golangci-lint run | `--color=never --show-stats=false --output.text.print-issued-lines=false` |
| staticcheck       | No flags needed (output is `file:line:col: message (CODE)`)               |
| gofmt             | `-l` (list files) or `-d` (diffs)                                         |
| goimports         | `-l` (list files) or `-d` (diffs)                                         |
| govulncheck       | No flags needed (text output is already clean)                            |

## go build

```bash
go build ./...
```

Go's compiler output is already AI-friendly — no colors, no progress bars, no banners. Errors print to stderr in `file:line:col: message` format, one per line.

- `-json` — emits build output as JSON (one event per line)

## go test

```bash
# Default — only failures and summary
go test ./...

# Verbose — shows all test names
go test -v ./...

# Structured JSON output
go test -json ./...
```

- `-v` — shows individual test names with PASS/FAIL status and `t.Log` output
- `-json` — NDJSON with `Action`, `Package`, `Test`, `Output` fields per event
- `-count=1` — disables test result caching (useful for re-runs)
- `-failfast` — stops after first test failure
- `-run regexp` — runs only matching tests
- `-short` — tells long-running tests to shorten their run time
- `-fullpath` — shows full file paths in error messages instead of relative

Single test: `go test -v -run TestName ./path/to/pkg`

Default output is already minimal — only failures and a one-line summary per package. No colors, no progress bars. The `-v` flag adds individual test names, which is useful for agents to understand what passed.

### Coverage

```bash
# Inline coverage percentage
go test -cover ./...

# Generate coverage profile
go test -coverprofile=cover.out ./...

# Function-level coverage report
go tool cover -func=cover.out
```

- `-cover` — appends coverage percentage to each package's summary line
- `-coverprofile=file` — writes coverage data to file for analysis
- `go tool cover -func=file` — shows per-function coverage in `file:line: funcName percentage` format

## go vet

```bash
go vet ./...
```

- `-json` — outputs JSON diagnostics
- `-fix` — applies automatic fixes instead of printing diagnostics
- `-fix -diff` — prints fixes as unified diffs instead of applying them (`-diff` requires `-fix`)

Default output is `file:line:col: message` — already compact. `go vet` runs automatically as part of `go test` (disable with `-vet=off`).

## go mod tidy

```bash
go mod tidy
```

Silent on success. Prints errors to stderr only when there are unresolvable dependency issues. No flags needed for AI agents.

- `-v` — prints information about removed modules to stderr

## golangci-lint

```bash
# Compact one-line-per-issue (no source code, no stats)
golangci-lint run --color=never --show-stats=false --output.text.print-issued-lines=false ./...

# Fix auto-fixable issues
golangci-lint run --fix --color=never ./...
```

- `--color=never` — disables ANSI colors (default is `auto`, which enables colors in TTY)
- `--show-stats=false` — removes the per-linter stats summary at the end
- `--output.text.print-issued-lines=false` — removes source code lines below each diagnostic (biggest savings)
- `--output.text.print-linter-name` — appends linter name in parentheses (default: true, useful for agents)
- `--output.text.colors=false` — disables colors in text output (alternative to `--color=never`)
- `--fix` — applies auto-fixes from linters and formatters that support it

Default output includes source code snippets and a stats summary — 2-3x more verbose than the compact form above. With `--output.text.print-issued-lines=false` and `--show-stats=false`, output is one line per issue: `file:line:col: message (linter-name)`.

Other output formats: `--output.json.path=stdout` (JSON), `--output.checkstyle.path=stdout`, `--output.sarif.path=stdout`, `--output.junit-xml.path=stdout`, `--output.code-climate.path=stdout`.

> **Note:** golangci-lint v2 (2025+) changed the output flag syntax from `--out-format` to `--output.<format>.path`. If using v1, use `--out-format=line-number` instead.

## staticcheck

```bash
staticcheck ./...
```

- `-f text` — default: `file:line:col: message (CODE)` — already compact
- `-f json` — NDJSON: one JSON object per diagnostic with `code`, `severity`, `location`, `message`
- `-f stylish` — grouped by file with decorative formatting and summary count (more verbose)
- `-checks "all,-ST1000"` — configure which checks to run

Default text output is already one of the cleanest in the Go ecosystem. No colors, no banners, one line per issue.

## gofmt

```bash
# Check which files need formatting
gofmt -l .

# Show diffs
gofmt -d .

# Fix formatting in place
gofmt -w .
```

- `-l` — lists files whose formatting differs from gofmt's (one path per line)
- `-d` — displays unified diffs without modifying files
- `-w` — rewrites files in place
- `-s` — simplify code (e.g., `s[a:len(s)]` to `s[a:]`)
- `-e` — report all errors (not just the first 10 on different lines)

gofmt produces no output when all files are correctly formatted. No colors, no banners.

## goimports

```bash
# Check which files need import fixes
goimports -l .

# Show diffs
goimports -d .

# Fix imports and formatting in place
goimports -w .
```

- `-l` — lists files that need changes
- `-d` — displays unified diffs
- `-w` — rewrites files in place
- `-local prefix` — puts imports beginning with prefix after third-party packages (e.g., `-local github.com/myorg`)

goimports is a superset of gofmt — it also adds missing imports and removes unused ones. Same output format as gofmt.

## govulncheck

```bash
# Scan for known vulnerabilities
govulncheck ./...
```

- `-format json` — structured JSON output with full vulnerability details and SBOM
- `-format sarif` — SARIF format for integration with security tools
- `-format openvex` — OpenVEX format
- `-scan module` — scan at module level (faster, less precise)
- `-scan package` — scan at package level (middle ground)
- `-scan symbol` — default: scan at symbol level (most precise, only reports vulns in code paths actually used)
- `-show traces` — shows call stacks from your code to the vulnerable function
- `-test` — include test files in the analysis

Default text output prints "No vulnerabilities found." on success, or a structured report with vulnerability ID, description, and affected packages on failure. No colors.

## AGENTS.md / CLAUDE.md Template

Copy this into your project's AGENTS.md or CLAUDE.md:

```markdown
## Running Tools

- Build: `go build ./...`
- Tests: `go test ./...`
- Tests (verbose): `go test -v ./...`
- Tests (single): `go test -v -run TestName ./path/to/pkg`
- Tests (no cache): `go test -count=1 ./...`
- Coverage: `go test -cover ./...`
- Vet: `go vet ./...`
- Lint: `golangci-lint run --color=never --show-stats=false --output.text.print-issued-lines=false ./...`
- Lint fix: `golangci-lint run --fix --color=never ./...`
- Static analysis: `staticcheck ./...`
- Format check: `gofmt -l .`
- Format fix: `gofmt -w .`
- Import check: `goimports -l .`
- Import fix: `goimports -w .`
- Vulnerability scan: `govulncheck ./...`
- Tidy deps: `go mod tidy`
```
