# Rust

## Quick Reference

| Tool               | AI-Friendly Flags                                                               |
|--------------------|---------------------------------------------------------------------------------|
| cargo build        | `--color=never -q` (progress bar auto-disables in non-TTY)                      |
| cargo check        | `--color=never -q --message-format=short`                                       |
| cargo test         | `--color=never -q -- --color=never`                                             |
| cargo clippy       | `--color=never -q --message-format=short`                                       |
| cargo clippy --fix | `--allow-dirty --color=never`                                                   |
| cargo fmt --check  | `--message-format=short -- --color=never`                                       |
| cargo fmt          | No flags needed (formats in place, no output on success)                        |
| cargo nextest run  | `--color=never --cargo-quiet --failure-output=immediate --success-output=never` |
| cargo audit        | `--format=json --color=never`                                                   |
| cargo deny check   | `--format=json --color=never`                                                   |

## Environment Variables

These apply to all `cargo` subcommands:

| Variable                   | Values                    | Effect                                   |
|----------------------------|---------------------------|------------------------------------------|
| `NO_COLOR=1`               | Any non-empty value       | Disables ANSI colors (since Rust 1.74.0) |
| `CARGO_TERM_COLOR`         | `auto`, `always`, `never` | Controls Cargo's color output            |
| `CARGO_TERM_PROGRESS_WHEN` | `auto`, `always`, `never` | Controls progress bar display            |
| `CARGO_TERM_QUIET`         | `true` / `false`          | Equivalent to `-q`                       |
| `CI`                       | Any value (presence-only) | Disables progress bar in `auto` mode     |
| `TERM=dumb`                | —                         | Disables progress bar in `auto` mode     |

Cargo auto-detects non-TTY environments: colors are stripped and the progress bar is disabled when stderr is not a terminal. Explicit flags are safer, but in many cases no flags are needed when piping output.

## cargo build

```bash
cargo build --color=never -q
```

- `--color=never` — disables ANSI colors (`auto` strips colors in non-TTY but explicit is safer)
- `-q` / `--quiet` — suppresses Compiling, Downloading, and Finished status lines (errors still print)
- `--message-format=json` — structured JSON to stdout, one object per line (`compiler-message`, `compiler-artifact`, `build-script-executed`, `build-finished`) — very verbose

## cargo check

```bash
cargo check --color=never -q --message-format=short
```

- `--message-format=short` — one-line-per-error format: `file:line:col: severity[code]: message`
- `--message-format=json` — structured JSON to stdout (same format as `cargo build`)
- `--color=never` — disables colors
- `-q` / `--quiet` — suppresses "Checking..." status lines

`cargo check` is faster than `cargo build` — it skips code generation and only runs the compiler frontend.

## cargo test

```bash
# Default — only failures and summary
cargo test --color=never -q -- --color=never

# Single test
cargo test --color=never -q test_name -- --color=never

# Exact match
cargo test --color=never -q test_name -- --color=never --exact
```

- `--color=never` — disables colors for Cargo output
- `-- --color=never` — disables colors for the test harness (libtest). **Must be passed separately after `--`** — Cargo's `--color` does not propagate to the test binary
- `-q` / `--quiet` — suppresses compilation status lines and shows one character per test instead of one line
- `-- --no-capture` — shows stdout/stderr from tests in real time (useful for debugging). `--nocapture` is a deprecated alias
- `-- --show-output` — shows stdout/stderr of passing tests after all tests complete
- `-- --test-threads=1` — serial execution for tests with shared state
- `--no-fail-fast` — continue running all test executables even if one fails

`--message-format=json` (Cargo flag) works on stable but only covers compilation messages. The test-results JSON format (`-- --format=json -Z unstable-options`) is unstable and requires nightly.

## cargo clippy

```bash
# Compact one-line-per-warning
cargo clippy --color=never -q --message-format=short

# Treat warnings as errors (common CI pattern)
cargo clippy --color=never -q --message-format=short -- -Dwarnings

# Auto-fix (--allow-dirty needed if working tree has uncommitted changes)
cargo clippy --fix --allow-dirty --color=never
```

- `--message-format=short` — one-line-per-diagnostic format (biggest token savings vs default)
- `--color=never` — disables colors
- `-q` / `--quiet` — suppresses "Checking..." status lines
- `-- -Dwarnings` — treats all warnings as errors (non-zero exit on any lint)
- `-- -Aclippy::lint_name` — suppress a specific lint
- `--fix` — automatically applies Clippy suggestions. Requires `--allow-dirty` if the working tree has uncommitted changes (common for AI agents)
- `--message-format=json` — structured JSON diagnostics

Default output includes multi-line code snippets with underlines and suggestions — significantly more verbose than `short`. The `short` format is 3-5x more token-efficient.

## cargo fmt

```bash
# Check which files need formatting (most token-efficient)
cargo fmt --check --message-format=short -- --color=never

# Fix formatting
cargo fmt
```

- `--check` — exits 1 if files need formatting, 0 if already formatted. Without `--message-format=short`, prints diffs of unformatted code
- `--message-format=short` — lists only file paths that need formatting, one per line (maps to rustfmt's `-l` flag)
- `--message-format=json` — JSON output via rustfmt (cannot be combined with `--check`)
- `-- --color=never` — disables colors. `--color` is a rustfmt flag, not a cargo-fmt flag — must be passed after `--`
- `-q` / `--quiet` — suppresses output

`cargo fmt` with no flags formats all files in place and produces no output on success.

## cargo nextest run

[cargo-nextest](https://nexte.st) is a popular third-party test runner with better output control than the built-in test harness.

```bash
# Failures only, no pass output, suppress compilation noise
cargo nextest run --color=never --cargo-quiet --failure-output=immediate --success-output=never

# Single test
cargo nextest run --color=never -E 'test(test_name)'
```

- `--color=never` — disables colors
- `--failure-output=immediate` — show failure output as it happens (also: `final`, `immediate-final`, `never`)
- `--success-output=never` — suppress output from passing tests (also: `immediate`, `final`, `immediate-final`)
- `--status-level=fail` — only show failed tests during execution (values: `none`, `fail`, `retry`, `slow`, `leak`, `pass`, `skip`, `all`)
- `--final-status-level=fail` — only show failures in the summary (values: `none`, `fail`, `flaky`, `slow`, `skip`, `pass`, `all`)
- `--cargo-quiet` — suppress Cargo compilation messages (repeat twice for full suppression)
- `--no-output-indent` — disable indentation of captured test output

nextest auto-detects non-TTY: in non-TTY mode, the progress bar switches to a line counter.

JUnit XML output is configured via `nextest.toml`, not CLI flags:

```toml
[profile.ci.junit]
path = "junit.xml"
store-success-output = false
store-failure-output = true
```

Run with `cargo nextest run --profile ci` to generate the report.

## cargo audit

```bash
cargo audit --format=json --color=never
```

- `--format=json` — structured JSON output (also accepts `sarif` for security tool integration)
- `--color=never` — disables colors
- `-q` / `--quiet` — suppresses non-essential output

Legacy shorthand: `--json` is equivalent to `--format=json`

## cargo deny

```bash
cargo deny --format=json --color=never check
```

Note: `--format` and `--color` are global flags and must appear before the `check` subcommand. Flags like `--hide-inclusion-graph` and `-s` belong to `check` and go after it.

- `--format=json` — each diagnostic as a single-line JSON object (also accepts `sarif` for security tool integration)
- `--color=never` — disables colors (no effect when `--format=json`)
- `--hide-inclusion-graph` — suppresses verbose inverse dependency graph in diagnostics (significant savings in human format)
- `-L error` / `--log-level=error` — only show errors (values: `off`, `error`, `warn`, `info`, `debug`, `trace`)
- `-s` / `--show-stats` — show statistics for all checks

## AGENTS.md / CLAUDE.md Template

Copy this into your project's AGENTS.md or CLAUDE.md:

```markdown
## Running Tools

- Build: `cargo build --color=never -q`
- Check: `cargo check --color=never -q --message-format=short`
- Tests: `cargo test --color=never -q -- --color=never`
- Tests (single): `cargo test --color=never -q test_name -- --color=never`
- Tests (nextest): `cargo nextest run --color=never --cargo-quiet --failure-output=immediate --success-output=never`
- Lint: `cargo clippy --color=never -q --message-format=short -- -Dwarnings`
- Lint fix: `cargo clippy --fix --allow-dirty --color=never`
- Format check: `cargo fmt --check --message-format=short -- --color=never`
- Format fix: `cargo fmt`
- Audit: `cargo audit --format=json --color=never`
- Deny: `cargo deny --format=json --color=never check`
```
