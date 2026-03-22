# Ruby

## Quick Reference

| Tool                    | AI-Friendly Flags                                                            |
|-------------------------|------------------------------------------------------------------------------|
| bundle install          | `--quiet`                                                                    |
| rake                    | `-s` (silent — suppresses rake's own messages, only shows task output)       |
| rspec                   | `--format progress --no-color --fail-fast`                                   |
| minitest                | `-q` (no dots) or `-v` (test names, no color by default)                     |
| rails test              | `--no-color --fail-fast`                                                     |
| rubocop                 | `--format emacs --no-color` (one-line-per-offense)                           |
| rubocop (fix)           | `-A --no-color` (safe + unsafe autocorrect)                                  |
| standard                | `--format emacs --no-color` (or `--format quiet` for silent-when-clean)      |
| standard (fix)          | `--fix --no-color`                                                           |
| srb tc                  | `--color never --no-error-count --no-error-sections`                         |
| reek                    | `--no-color --no-progress -s`                                                |
| brakeman                | `-q --no-progress --no-pager --no-color`                                     |
| rails generate          | `--quiet --force` (suppresses status lines, prevents prompts)                |
| rails db:migrate        | `VERBOSE=false` (suppresses migration output)                                |
| erb_lint                | `-f compact` (or `-f json` for structured output)                            |

## bundle

```bash
# Quiet install (warnings and errors only)
bundle install --quiet

# Verbose but clean (parallel, retries)
bundle install --jobs=4 --retry=3
```

- `--quiet` — suppresses progress output, only shows warnings and errors
- `--jobs=N` / `-jN` — parallel gem downloads/installs
- `--retry=N` — retry failed network requests (default: 3)
- `--local` — use only locally cached gems, skip all network requests

`bundle update` and `bundle add` also accept `--quiet`.

Bundler auto-detects non-TTY and disables colors when stdout is not a terminal. The `--no-color` global flag is available on all `bundle` subcommands as a safety net.

Env var equivalents: `NO_COLOR=1` (disables color), `BUNDLE_JOBS=4`, `BUNDLE_RETRY=3`, `BUNDLE_IGNORE_MESSAGES=true` (suppresses post-install messages).

## rake

```bash
# Silent — only shows output from the task itself
rake -s taskname

# Quiet — suppresses rake's own logging
rake -q taskname
```

- `-s` / `--silent` — suppresses rake's own messages including "in directory" announcements
- `-q` / `--quiet` — suppresses rake's log messages but keeps directory announcements
- `-t` / `--trace` — shows full backtrace on error (trace output goes to stderr by default)

Rake itself produces no colored output — any colors come from the tasks being run (test frameworks, linters, etc.).

## rspec

```bash
# Compact output (default dots, no color)
rspec --format progress --no-color

# One line per failure (most token-efficient for failures)
rspec --format failures --no-color --fail-fast

# Structured JSON output
rspec --format json --no-color
```

- `--format progress` — default: dots for pass, `F` for fail, `*` for pending
- `--format failures` — one line per failure: `spec/foo_spec.rb:42:should do something` (extremely compact)
- `--format json` — full structured JSON with test details, timing, and exception info
- `--format documentation` — nested describe/it names (verbose but readable)
- `--no-color` — disables ANSI color codes
- `--fail-fast` — stop after first failure (or `--fail-fast=N` after N failures)
- `--order defined` — deterministic run order

Single test: `rspec spec/models/user_spec.rb:42` or `rspec -e "validates email"`

RSpec auto-detects non-TTY and disables color by default, unless `--color` is set in `.rspec`. `--no-color` overrides all.

Env var: `SPEC_OPTS="--format json --no-color --fail-fast"` injects options on every run without modifying project config.

Other formatters: `html`. No `--quiet` flag — use `--format failures` for minimal output.

## minitest

```bash
# Quiet (no progress dots)
ruby -Ilib:test test/my_test.rb -q

# Verbose (each test name printed)
ruby -Ilib:test test/my_test.rb -v

# Via rake
rake test
```

- `-q` / `--quiet` — suppresses progress dots
- `-v` / `--verbose` — prints each test name as it runs
- `-n PATTERN` / `--include` — run specific test: `-n test_something` or `-n /regex/`
- `-s SEED` / `--seed` — set random seed for ordering

Minitest has no color by default. The bundled pride plugin (`-p`) or the minitest-reporters gem add color — both auto-detect TTY and disable color when piped.

For structured output, use the minitest-reporters gem with `JUnitReporter` for XML.

Env vars: `A="--verbose"` passes options through `rake test` when using `Minitest::TestTask` (the older `Rake::TestTask` still uses `TESTOPTS`).

## rails test

```bash
# Minimal output
bin/rails test --no-color --fail-fast

# Single test by file and line
bin/rails test test/models/user_test.rb:42 --no-color --fail-fast

# Deferred output (failures printed after full run)
bin/rails test --no-color --fail-fast -d
```

- `--no-color` — disables ANSI colors
- `--fail-fast` / `-f` — stop after first failure
- `-d` / `--defer-output` — collects failures, prints them after the run completes
- `-v` / `--verbose` — shows per-test names and timing
- `-b` / `--backtrace` — shows full unfiltered backtrace (verbose, use selectively)
- `-n PATTERN` / `--include` — filter tests by name

Rails auto-detects non-TTY and disables color when output is piped.

## rails generate

```bash
bin/rails generate model User name:string --quiet --force
```

- `-q` / `--quiet` — suppresses all "create", "invoke" status lines
- `-f` / `--force` — overwrites existing files without prompting (prevents hanging)
- `--skip-collision-check` — skips class-name collision checking (file collisions are handled by `--force`)

Without `--quiet`, generators print a status line for every file created or modified.

## rails db:migrate

```bash
VERBOSE=false bin/rails db:migrate
```

- `VERBOSE=false` — suppresses all migration output (create_table, add_column, etc.)
- `--trace` — shows full rake task chain and backtrace on error

There is no `--quiet` flag for migrations — use the `VERBOSE` env var.

## rubocop

```bash
# One-line-per-offense (most token-efficient text format)
rubocop --format emacs --no-color

# Structured JSON output
rubocop --format json --no-color

# Silent when clean, simple format when offenses found
rubocop --format quiet --no-color

# Auto-fix (safe corrections only)
rubocop -a --no-color

# Auto-fix (safe + unsafe corrections)
rubocop -A --no-color
```

- `--format emacs` — `file:line:col: severity: message` (one line per offense, easily parseable)
- `--format json` — full structured JSON with metadata, offenses, and summary
- `--format quiet` — silent when clean, simple format when offenses exist
- `--format offenses` — violation counts per cop (overview)
- `--no-color` — disables ANSI colors
- `-a` / `--autocorrect` — safe corrections only
- `-A` / `--autocorrect-all` — safe and unsafe corrections
- `--display-only-fail-level-offenses` — only show offenses at or above `--fail-level`
- `--fail-level SEVERITY` — minimum severity for non-zero exit (`A`, `I`, `R`, `C`, `W`, `E`, `F`)
- `-F` / `--fail-fast` — stop after first file with offenses

RuboCop auto-detects non-TTY via the Rainbow gem and disables color when piped.

Env var: `RUBOCOP_OPTS="--format emacs --no-color"` injects options on every run.

Other formatters: `simple`, `clang` (with source context), `progress` (default), `tap`, `junit`, `markdown`, `github`, `html`, `files`, `worst`, `fuubar`, `pacman`, `autogenconf`.

## standard

```bash
# One-line-per-offense
standardrb --format emacs --no-color

# Silent when clean
standardrb --format quiet --no-color

# Auto-fix (safe)
standardrb --fix --no-color

# Auto-fix (safe + unsafe)
standardrb --fix-unsafely --no-color
```

- `--format FORMAT` — accepts all RuboCop formatters (`emacs`, `json`, `quiet`, `progress`, etc.)
- `--no-color` — disables ANSI colors
- `--fix` — safe automatic fixes (equivalent to RuboCop's `-a`)
- `--fix-unsafely` — all fixes including potentially behavior-changing ones (equivalent to RuboCop's `-A`)
- `--generate-todo` — creates `.standard_todo.yml` listing files with errors

Standard wraps RuboCop with a fixed, curated ruleset — no per-cop configuration. Configuration is via `.standard.yml`.

Env var: `STANDARDOPTS="--format emacs --no-color"` passes options when running via `rake standard`.

## srb tc

```bash
# Clean one-line-per-error output
srb tc --color never --no-error-count --no-error-sections

# Quiet (non-critical errors only)
srb tc --quiet --color never
```

- `--color never` — disables ANSI colors (`auto` detects TTY on stderr)
- `--no-error-count` — removes the trailing "Errors: N" summary line
- `--no-error-sections` — prints only the first line of each error (removes multi-line context)
- `-q` / `--quiet` — silences all non-critical errors
- `--suppress-error-code CODE` — exclude specific error codes (repeatable)
- `--isolate-error-code CODE` — include only specific error codes (repeatable)

Sorbet has no JSON format for error output. Default output is `file:line: message` which is already compact.

## reek

```bash
# Single-line format, no noise
reek --no-color --no-progress -s lib/

# Structured JSON output
reek --no-color --no-progress -f json lib/
```

- `-s` / `--single-line` — one line per smell, editor-compatible format
- `--no-color` — disables colored output
- `-P` / `--no-progress` — disables per-file progress indicator
- `-f FORMAT` — output format: `text` (default), `json`, `yaml`, `xml`, `html`, `github`
- `--no-documentation` — hides links to documentation pages
- `--smell SMELL` — filter to specific smell detector (repeatable)

Reek has no `--quiet` flag. Use `-s` (single-line) for the most compact text output.

## brakeman

```bash
# Quiet JSON report (most token-efficient)
brakeman -q --no-progress --no-pager --no-color -f json

# Quiet text report
brakeman -q --no-progress --no-pager --no-color

# Summary only
brakeman -q --no-progress --no-pager --no-color --summary
```

- `-q` / `--quiet` — suppresses informational messages, only outputs the report
- `--no-progress` — disables progress indicator
- `--no-pager` — outputs directly without paging (pager is on by default)
- `--no-color` — disables ANSI colors
- `-f FORMAT` — output format: `text` (default), `json`, `csv`, `markdown`, `junit`, `sarif`, `github`, `html`, `tabs`, `table`, `pdf`, `codeclimate`/`cc`, `plain`, `sonar`
- `--summary` — only output warning summary
- `-w N` — filter by confidence level: `1` (all), `2` (medium+), `3` (high only)
- `--absolute-paths` — use absolute file paths in reports

All Brakeman output except the report is sent to stderr.

## erb_lint

```bash
# Compact one-line-per-offense
bundle exec erb_lint -f compact --lint-all

# Structured JSON output
bundle exec erb_lint -f json --lint-all

# Auto-fix
bundle exec erb_lint -f compact --lint-all -a
```

- `-f FORMAT` / `--format` — output format: `multiline` (default), `compact`, `json`, `junit`, `gitlab`
- `--lint-all` — lint all files matching glob (default: `**/*.html{+*,}.erb`)
- `-a` / `--autocorrect` — auto-fix violations
- `--enable-all-linters` — enable every available linter
- `--enable-linters L1,L2` — enable only specific linters

ERB Lint has no `--no-color` or `--quiet` flags. The `compact` format is already clean.

## AGENTS.md / CLAUDE.md Template

Copy this into your project's AGENTS.md or CLAUDE.md:

```markdown
## Running Tools

- Install: `bundle install --quiet`
- Run task: `rake -s taskname`
- Tests (Minitest): `ruby -Ilib:test test/my_test.rb -q`
- Tests (RSpec): `rspec --format progress --no-color --fail-fast`
- Tests (RSpec single): `rspec spec/models/user_spec.rb:42 --no-color --fail-fast`
- Tests (Rails): `bin/rails test --no-color --fail-fast`
- Tests (Rails single): `bin/rails test test/models/user_test.rb:42 --no-color --fail-fast`
- Lint: `rubocop --format emacs --no-color`
- Lint fix: `rubocop -A --no-color`
- Lint (Standard): `standardrb --format emacs --no-color`
- Lint fix (Standard): `standardrb --fix --no-color`
- Type check: `srb tc --color never --no-error-count --no-error-sections`
- Code smells: `reek --no-color --no-progress -s lib/`
- Security scan: `brakeman -q --no-progress --no-pager --no-color`
- ERB lint: `bundle exec erb_lint -f compact --lint-all`
- ERB lint fix: `bundle exec erb_lint -f compact --lint-all -a`
- Generate: `bin/rails generate model User name:string --quiet --force`
- Migrate: `VERBOSE=false bin/rails db:migrate`
```
