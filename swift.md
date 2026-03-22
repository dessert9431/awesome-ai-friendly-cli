# Swift

## Quick Reference

| Tool                    | AI-Friendly Flags                                                                       |
|-------------------------|-----------------------------------------------------------------------------------------|
| swift build             | `--no-color-diagnostics -q`                                                             |
| swift test              | `--no-color-diagnostics -q`                                                             |
| swift package           | `--no-color-diagnostics -q` (`show-dependencies --format json`, `describe --type json`) |
| xcodebuild              | `-quiet` (no color by default; `-list -json` for JSON queries)                          |
| xcodebuild + xcbeautify | `2>&1 \| xcbeautify --disable-colored-output -q`                                        |
| swiftlint               | `--quiet --reporter json`                                                               |
| swiftlint --fix         | `--quiet`                                                                               |
| swiftformat --lint      | `--quiet --reporter json`                                                               |
| swiftformat             | `--quiet` (formats in place, no output on success)                                      |
| swift-format lint       | `--no-color-diagnostics`                                                                |
| swift-format format     | `--in-place` (no flags needed for quiet output)                                         |
| periphery scan          | `--quiet --format json --disable-update-check`                                          |
| xcpretty                | `--no-color`                                                                            |

## Environment Variables

These apply to all Swift Package Manager commands (`swift build`, `swift test`, `swift package`):

| Variable     | Effect                                           |
|--------------|--------------------------------------------------|
| `NO_COLOR=1` | Disables color diagnostics (any non-empty value) |

SPM auto-detects non-TTY environments: color diagnostics are disabled when not connected to a terminal. Explicit `--no-color-diagnostics` is safer for scripted use.

xcodebuild does not use ANSI colors by default, so `NO_COLOR` has no effect on it.

xcbeautify respects `NO_COLOR=1` to disable colored output.

## swift build

```bash
swift build --no-color-diagnostics -q
```

- `--no-color-diagnostics` — disables ANSI color codes in compiler diagnostics (auto-disabled in non-TTY, but explicit is safer)
- `-q` / `--quiet` — suppresses all output except errors

## swift test

```bash
# Default — errors only
swift test --no-color-diagnostics -q

# Single test
swift test --no-color-diagnostics -q --filter 'TestClassName/testMethodName'

# Skip specific tests
swift test --no-color-diagnostics -q --skip 'PerformanceTests'

# Generate xUnit XML report
swift test --no-color-diagnostics -q --xunit-output test-results.xml
```

- `--no-color-diagnostics` — disables color in compiler diagnostics
- `-q` / `--quiet` — suppresses all output except errors
- `--filter <regex>` — run only tests matching the pattern (format: `TestTarget.TestCase/testMethod`)
- `--skip <regex>` — skip tests matching the pattern
- `--xunit-output <path>` — generates xUnit XML report at the specified path
- `--parallel` / `--no-parallel` — enable/disable parallel test execution (default: `--no-parallel`)
- `-l` / `--list-tests` — lists test methods in specifier format without running them

## swift package

```bash
# Show dependencies as JSON
swift package show-dependencies --format json --no-color-diagnostics -q

# Describe package as JSON
swift package describe --type json --no-color-diagnostics -q

# Dump Package.swift as JSON
swift package dump-package
```

- `--no-color-diagnostics` — disables color in diagnostics
- `-q` / `--quiet` — suppresses all output except errors
- `show-dependencies --format <format>` — output format for dependency graph (values: `text`, `dot`, `json`, `flatlist`; default: `text`)
- `describe --type <type>` — output format for package description (values: `json`, `text`, `mermaid`; default: `text`)
- `dump-package` — always outputs Package.swift as JSON to stdout

## xcodebuild

```bash
# Build — warnings and errors only
xcodebuild -scheme MyScheme -quiet

# Test — warnings and errors only
xcodebuild test -scheme MyScheme -destination 'platform=iOS Simulator,name=iPhone 16' -quiet

# List targets/schemes as JSON
xcodebuild -list -json

# Enumerate tests as JSON
xcodebuild -enumerate-tests -scheme MyScheme -test-enumeration-format json -test-enumeration-output-path -
```

- `-quiet` — suppresses all output except warnings and errors
- `-json` — outputs as JSON for query commands (`-list`, `-showBuildSettings`, `-showsdks`, `-showComponent`); **implies `-quiet`**. Not available for build or test actions
- `-hideShellScriptEnvironment` — hides environment variables from shell script build phases (reduces noise in build logs)
- `-resultBundlePath <path>` — writes structured result bundle for programmatic access
- `-test-enumeration-format json` — JSON output for `-enumerate-tests` (use `-test-enumeration-output-path -` to write to stdout)
- `-dry-run` — runs everything except actual commands

xcodebuild does not emit ANSI colors by default — no `--no-color` flag is needed.

### Piping through xcbeautify

Raw xcodebuild output is extremely verbose. Pipe through [xcbeautify](https://github.com/cpisciotta/xcbeautify) for cleaner output:

```bash
# Build with formatted output
xcodebuild -scheme MyScheme 2>&1 | xcbeautify --disable-colored-output -q

# Test with only failures shown
xcodebuild test -scheme MyScheme -destination 'platform=iOS Simulator,name=iPhone 16' 2>&1 | xcbeautify --disable-colored-output --quieter

# For parallel testing, set NSUnbufferedIO to avoid garbled output
NSUnbufferedIO=YES xcodebuild test -scheme MyScheme -destination 'platform=iOS Simulator,name=iPhone 16' 2>&1 | xcbeautify --disable-colored-output -q
```

- `--disable-colored-output` — disables ANSI colors
- `-q` / `--quiet` — only prints tasks with warnings or errors
- `--quieter` / `-qq` — only prints tasks with errors
- `--is-ci` — includes test results even under `--quiet` / `--quieter`
- `--disable-logging` — suppresses xcbeautify's own startup information table
- `--renderer <renderer>` — output renderer (values: `terminal`, `github-actions`, `teamcity`, `azure-devops-pipelines`; auto-detects from `GITHUB_ACTIONS`, `TEAMCITY_VERSION`, `AZURE_DEVOPS_PIPELINES` env vars)

### Piping through xcpretty

[xcpretty](https://github.com/xcpretty/xcpretty) is an older alternative to xcbeautify:

```bash
set -o pipefail && xcodebuild -scheme MyScheme 2>&1 | xcpretty --no-color
```

- `--no-color` — disables colored output (auto-detects TTY by default)
- `-s` / `--simple` — simple output (default)
- `-t` / `--test` — RSpec-style output
- `--tap` — Test Anything Protocol output
- `-r junit` / `-r html` / `-r json-compilation-database` — generate JUnit XML, HTML, or JSON compilation database report

`set -o pipefail` is required so the pipeline returns xcodebuild's exit code, not xcpretty's.

## swiftlint

```bash
# Lint with JSON output
swiftlint --quiet --reporter json

# Lint single file
swiftlint lint --quiet --reporter json path/to/file.swift

# Auto-fix violations
swiftlint --fix --quiet
```

- `--quiet` — suppresses status logs (e.g., 'Linting <file>', 'Done linting'); violations still print normally
- `--reporter <reporter>` — output format (values: `xcode`, `json`, `csv`, `checkstyle`, `codeclimate`, `emoji`, `github-actions-logging`, `gitlab`, `html`, `junit`, `markdown`, `relative-path`, `sarif`, `sonarqube`, `summary`; default: `xcode`)
- `--strict` — upgrades warnings to errors
- `--lenient` — downgrades errors to warnings
- `--silence-deprecation-warnings` — hides rule deprecation warnings
- `--progress` — shows a live-updating progress bar (omit for AI agents)
- `--fix` — auto-fixes correctable violations

The default `xcode` reporter uses the `file:line:col: severity: message` format, which is already compact. Use `--reporter json` for structured output.

## swiftformat

```bash
# Check formatting (lint mode)
swiftformat --lint --quiet --reporter json .

# Fix formatting
swiftformat --quiet .

# Dry run — show what would change
swiftformat --dry-run --quiet .
```

- `--quiet` — suppresses non-error output
- `--lint` — checks formatting without modifying files (exits non-zero on violations)
- `--reporter <reporter>` — output format for `--lint` mode (values: `json`, `github-actions-log`, `xml`, `sarif`)
- `--dry-run` — shows what would change without modifying files
- `--strict` — treats violations as errors
- `--lenient` — treats violations as warnings

SwiftFormat auto-detects TTY for stderr colorization. In non-TTY mode, color is automatically disabled.

## swift-format

[swift-format](https://github.com/swiftlang/swift-format) is Apple's official formatter (distinct from Nick Lockwood's SwiftFormat above).

```bash
# Check formatting
swift-format lint --no-color-diagnostics -r .

# Fix formatting
swift-format format --in-place -r .
```

- `--no-color-diagnostics` — disables ANSI color in diagnostics
- `-r` / `--recursive` — processes `.swift` files in directories recursively
- `--in-place` / `-i` — overwrites files in place (for `format` subcommand)
- `--strict` / `-s` — treats all findings as errors (for `lint` subcommand)
- `-p` / `--parallel` — processes files across multiple cores
- `--ignore-unparsable-files` — skips files with syntax errors instead of reporting them

No `--quiet`, `--reporter`, or `--format` flag exists. Output is minimal by default — `lint` prints diagnostics in `file:line:col: warning/error: message` format, `format` outputs formatted source to stdout (or modifies in place with `-i`).

## periphery

```bash
periphery scan --quiet --format json --disable-update-check
```

- `--quiet` — suppresses all logging except results
- `--format <format>` — output format (values: `xcode`, `csv`, `json`, `checkstyle`, `codeclimate`, `github-actions`, `github-markdown`, `gitlab-codequality`; default: `xcode`)
- `--disable-update-check` — skips checking for newer versions (avoids network call and extra output)
- `--strict` — exits with non-zero status if any unused code is found
- `--skip-build` — skips the project build step (use when already built)
- `--relative-results` — outputs file paths relative to current directory

The default `xcode` reporter uses `file:line:col: warning: message` format.

## AGENTS.md / CLAUDE.md Template

Copy this into your project's AGENTS.md or CLAUDE.md:

```markdown
## Running Tools

- Build: `swift build --no-color-diagnostics -q`
- Tests: `swift test --no-color-diagnostics -q`
- Tests (single): `swift test --no-color-diagnostics -q --filter 'TestClassName/testMethodName'`
- Lint: `swiftlint --quiet --reporter json`
- Lint fix: `swiftlint --fix --quiet`
- Format check: `swiftformat --lint --quiet --reporter json .`
- Format fix: `swiftformat --quiet .`
- Dead code: `periphery scan --quiet --format json --disable-update-check`
- xcodebuild: `xcodebuild -scheme MyScheme -quiet`
- xcodebuild (pretty): `xcodebuild -scheme MyScheme 2>&1 | xcbeautify --disable-colored-output -q`
```
