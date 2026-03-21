# Python

## Quick Reference

| Tool                     | AI-Friendly Flags                                                                           |
|--------------------------|---------------------------------------------------------------------------------------------|
| pip install              | `-q --no-color --disable-pip-version-check --no-input --progress-bar off`                   |
| uv pip install / uv sync | `--quiet --color never`                                                                     |
| poetry install           | `--quiet --no-ansi --no-interaction`                                                        |
| pytest                   | `--no-header --tb=short -q --color=no`                                                      |
| ruff check               | `--output-format=concise` (or `--statistics` for overview)                                  |
| ruff format              | `--check -q`                                                                                |
| mypy                     | `--no-color-output --no-error-summary --no-pretty --show-column-numbers --show-error-codes` |
| pyright                  | No flags needed (default output is already clean)                                           |
| black                    | `--check --diff --no-color`                                                                 |
| flake8                   | No flags needed (default output is already `file:line:col: CODE message`)                   |
| pylint                   | `--score=no`                                                                                |
| isort                    | `--check --diff`                                                                            |
| bandit                   | `-f custom --msg-template "{abspath}:{line}: {test_id}: {msg} [{severity}:{confidence}]"`   |
| coverage report          | `--format=total` (just the number) or `--format=markdown --skip-covered --skip-empty -m`    |
| tox                      | `--colored no -q`                                                                           |
| python manage.py test    | `--no-color --verbosity 0 --no-input`                                                       |
| flask run                | `--no-debugger --no-reload`                                                                 |

## pip

```bash
# Minimal output (warnings and errors only)
pip install -q --no-color --disable-pip-version-check --no-input

# Silent install (errors only)
pip install -qq --no-color --disable-pip-version-check --no-input

# Verbose but clean (no progress bar, no noise)
pip install --no-color --no-input --progress-bar off --disable-pip-version-check

# List packages (structured)
pip list --format=json --no-color --disable-pip-version-check
```

- `-q` — warnings and errors only. Stackable: `-qq` errors only, `-qqq` critical only
- `--no-color` — disables ANSI color codes
- `--progress-bar off` — disables download progress bar (`auto` suppresses it with `-q`)
- `--no-input` — disables interactive prompts (prevents hanging)
- `--disable-pip-version-check` — suppresses "[notice] A new release of pip is available" noise

For structured install output: `pip install --report - -q` generates JSON to stdout with resolved packages, versions, and metadata.

`pip list` formats: `json` (machine-parseable), `freeze` (requirements.txt style), `columns` (default, padded table).

Env var equivalents: `PIP_NO_COLOR=1`, `PIP_PROGRESS_BAR=off`, `PIP_NO_INPUT=1`, `PIP_DISABLE_PIP_VERSION_CHECK=1`.

## uv

```bash
# Silent install
uv pip install <pkg> --quiet --color never

# Verbose but clean (summary only, no progress bar)
uv pip install <pkg> --no-progress --color never

# List packages (structured)
uv pip list --format=json --quiet --color never

# Sync from lockfile (structured)
uv sync --quiet --color never
```

- `--quiet` / `-q` — suppresses all output. Stackable
- `--no-progress` — disables progress bars but keeps summary text
- `--color never` — disables ANSI colors

uv is fast and already quiet by default — a simple install prints a few summary lines. A single `-q` silences it completely. uv never prompts interactively — no `--no-input` needed.

`uv pip list` formats: `json`, `freeze`, `columns` (same as pip).

Env var equivalents: `UV_NO_PROGRESS=1`, `UV_COLOR=never`.

## poetry

```bash
# Silent install
poetry install --quiet --no-ansi --no-interaction

# Verbose but clean
poetry install --no-ansi --no-interaction

# List packages (structured)
poetry show --format json --no-ansi
```

- `--quiet` / `-q` — suppresses all output
- `--no-ansi` — disables colors and ANSI formatting
- `--no-interaction` / `-n` — disables interactive prompts

Poetry's default output is already minimal ("Installing dependencies from lock file"). A single `-q` silences it completely.

Env var equivalents: `POETRY_NO_INTERACTION=1`.

## pytest

```bash
# Recommended for AI agents
pytest --no-header --tb=short -q --color=no

# Maximum compression (one line per failure)
pytest --no-header --tb=line -q --color=no

# Just pass/fail (no tracebacks)
pytest --no-header --tb=no -q --color=no
```

- `--tb=short` — shortened tracebacks: file:line, failing code line, error message (~38% savings on failures)
- `--tb=line` — one line per failure: `file:line: ExceptionType: message` (~57% savings)
- `--tb=no` — no tracebacks at all, only summary lines (~79% savings)
- `-q` — quiet: removes progress dots, shows compact summary
- `--no-header` — removes "platform darwin — Python 3.x, pytest-x.x" header block
- `--color=no` — disables ANSI colors

Other useful flags:

- `--no-summary` — removes "short test summary info" block (loses failure file paths — use with caution)
- `--disable-warnings` — suppresses deprecation warnings section
- `-x` / `--exitfirst` — stop on first failure (useful for iterative fixing)
- `--override-ini="console_output_style=count"` — shows `[11/17]` instead of `[64%]`

Single test: `pytest --no-header --tb=short -q --color=no tests/test_file.py::test_name`

`-qq` is nearly identical to `-q` — only removes the final status line ("17 passed in 0.05s"). The savings are negligible.

For structured output: `pip install pytest-json-report`, then `pytest --json-report --json-report-file=report.json`.

## ruff

```bash
# One-line-per-error format (68% smaller than default)
ruff check src/ --output-format=concise

# Quick overview of violation counts (91% smaller)
ruff check src/ --statistics

# Fix all auto-fixable issues
ruff check src/ --fix --unsafe-fixes

# Check formatting (just list files needing changes)
ruff format --check -q src/

# Show formatting diff
ruff format --check --diff src/
```

- `--output-format=concise` — one line per violation: `file:line:col: CODE message` (best for AI)
- `--output-format=grouped` — violations grouped by file with a file header (easier scanning when many files have errors)
- `--output-format=full` — default: includes source context and help text (verbose)
- `--statistics` — rule counts only: `7 F401 [*] unused-import` (most token-efficient overview)
- `-q` — suppresses non-diagnostic messages (does NOT reduce violation output)
- `-s` / `--silent` — suppresses ALL output, exit code only

Other formats: `json` (very verbose, includes fix edits), `json-lines`, `pylint`, `github`, `gitlab`, `azure`, `junit`, `rdjson`, `sarif`.

The default `full` format includes source code snippets and help text per violation — can be 5-15x larger than `concise` depending on violation density. The `concise` format is the best balance of information and token efficiency.

For `ruff format`: `--check -q` just lists files needing formatting (minimal output). `--check --diff` shows unified diffs of what would change.

> **Note:** `--output-format=text` was renamed to `full` in ruff 0.9+. Use `concise` for the most compact single-line format.

## mypy

```bash
# Clean one-line-per-error output
mypy src/ --no-color-output --no-error-summary --no-pretty --show-column-numbers --show-error-codes

# JSON output (NDJSON, one object per line)
mypy src/ --output json
```

- `--no-pretty` — disables source code snippets and `^` markers (biggest savings)
- `--no-color-output` — disables ANSI colors
- `--no-error-summary` — removes trailing "Found N errors in M files" line
- `--show-column-numbers` — adds column number: `file:line:col: error: message`
- `--show-error-codes` — shows `[error-code]` suffix (on by default in modern mypy, but explicit is safer)
- `--hide-error-context` — removes "note:" context lines added by `--show-error-context` (context is off by default, so this flag is only needed if your config enables it)

The `--output json` format produces NDJSON (one JSON object per line) with `file`, `line`, `column`, `message`, `code`, `severity` fields. Note: JSON output may consolidate or drop some context-dependent diagnostics.

mypy auto-detects non-TTY and disables colors, but `--no-color-output` is explicit and safer. The `--no-pretty` flag is the most impactful — it removes multi-line source snippets that significantly inflate output.

## pyright

```bash
# Default output is already clean
pyright src/

# Errors only (suppress warnings)
pyright src/ --level error

# Structured JSON output
pyright src/ --outputjson
```

- Default output already includes `file:line:col`, severity, and rule name in parentheses
- `--level error` — shows only errors, suppresses warnings and informational messages
- `--outputjson` — full JSON with character ranges (`start.line`, `start.character`, `end.line`, `end.character`), severity, and rule name

Pyright has no `--no-color`, `--quiet`, or `--format` flags. Color is automatically disabled in non-TTY environments (which is the case for AI agent pipes). No additional flags needed for clean output.

> **Note:** JSON output uses 0-based line and character numbers. Add 1 to convert to the 1-based numbers shown in text output.

## black

```bash
# Check what needs formatting (with diffs)
black --check --diff src/

# Check only (exit code, no output)
black --check -q src/

# Fix formatting
black src/
```

- `--check` — dry-run, reports which files would change (exit code 1 if changes needed)
- `--diff` — shows unified diff of changes without modifying files (combine with `--check` for non-zero exit code on differences)
- `-q` / `--quiet` — zero output, exit code only
- `--no-color` — strips ANSI from diff output (only relevant with `--diff`)

Black auto-detects non-TTY and disables color. The default `--check` output lists files that would change — `--diff` adds actionable diffs. `-q` produces zero output and relies entirely on exit code.

Black is a fixer — use it without `--check` to apply changes directly.

## flake8

```bash
# Default output is already optimal
flake8 src/

# Filenames only
flake8 src/ -q

# No output, exit code only
flake8 src/ -qq
```

- Default format: `file:line:col: CODE message` — already compact and parseable
- `-q` — filenames with errors only (one per line)
- `-qq` — no output, exit code only
- `--color=never` — disables colors (already off in non-TTY, but explicit is safer)
- `--statistics` — appends violation counts per error code at the end (adds ~36% more output)

Other formats: `pylint` (drops column numbers — less useful), `default` (same as no flag).

flake8's default output is already one of the most AI-friendly formats in the Python ecosystem. No additional flags needed for most workflows.

## pylint

```bash
# Default text format (most compact)
pylint src/ --score=no

# Errors only (96% reduction)
pylint src/ --errors-only

# Machine-readable (parseable)
pylint src/ --output-format=parseable --score=no
```

- `--score=no` — removes the "Your code has been rated at X/10" line
- `--errors-only` / `-E` — only E-level messages, suppresses warnings and conventions (~96% reduction)
- `--output-format=parseable` — `file:line: [CODE(symbol)] message` (slightly longer than default text)
- `--output-format=text` — default format: `file:line:col: CODE: message (symbol)`

Avoid: `--output-format=json` (3.4x larger), `--output-format=json2` (5x larger), `--output-format=colorized` (adds ANSI codes). JSON formats are massively token-wasteful for pylint.

> **Note:** pylint has no `-q` / `--quiet` flag. Use `--score=no` and `--errors-only` to control output verbosity.

## isort

```bash
# Check with diffs (shows what would change)
isort --check --diff src/

# Check only (which files have issues)
isort --check src/

# Fix imports
isort src/
```

- `--check` / `-c` — dry-run, reports files with incorrect import sorting
- `--diff` / `--df` — shows unified diff of import reordering
- `-q` / `--quiet` — suppresses "Skipping" and success messages (does NOT suppress error output)

Color is opt-in (`--color`), not on by default — good for AI out of the box.

isort is a fixer — use it without `--check` to apply changes directly. Pair with `--profile black` (or set in config) to match black formatting.

## bandit

```bash
# One-line-per-issue format (71% reduction)
bandit -r src/ -f custom --msg-template "{abspath}:{line}: {test_id}: {msg} [{severity}:{confidence}]"

# CSV format (46% reduction)
bandit -r src/ -f csv

# Medium+ severity only
bandit -r src/ -ll -f custom --msg-template "{abspath}:{line}: {test_id}: {msg} [{severity}:{confidence}]"
```

- `-f custom --msg-template "..."` — custom one-line-per-issue format (most token-efficient)
- `-f csv` — compact CSV rows with all metadata
- `-ll` — only medium and high severity (filters low-severity noise)
- `-lll` — only high severity
- `-q` — removes informational header lines (minimal savings, ~5%)
- `-r` — recursive scan

Default output includes multi-line code snippets, CWE links, and a metrics summary — very verbose. The custom format with `--msg-template` is 71% smaller and gives one parseable line per issue.

Avoid: `-f json` (1.6x larger than default due to embedded code snippets), `-f screen` (adds ANSI codes).

## coverage

```bash
# Just the total percentage
coverage report --format=total

# Markdown table (good for LLM consumption)
coverage report --format=markdown --skip-covered --skip-empty -m

# JSON report (structured)
coverage json -q --pretty-print

# Run tests with coverage
coverage run -m pytest tests/
```

- `--format=total` — outputs only the total coverage percentage (e.g., `90`)
- `--format=markdown` — markdown table with file-by-file breakdown
- `--skip-covered` — hides files with 100% coverage
- `--skip-empty` — hides files with no executable code
- `-m` / `--show-missing` — shows line numbers of uncovered code
- `--fail-under=MIN` — exit code 2 if coverage below threshold

coverage output has no ANSI colors by default — no `--no-color` flag needed.

For programmatic use: `coverage json` generates a full JSON report with per-file, per-function data, `missing_lines`, `executed_lines`, and `summary`.

## tox

```bash
# Minimal output
tox --colored no -q

# Parallel execution (no spinner)
tox run-parallel --colored no -p auto --parallel-no-spinner

# Structured results
tox --colored no --result-json results.json
```

- `--colored no` — disables colors
- `-q` — decreases verbosity (stackable: `-qq` for errors only)
- `--parallel-no-spinner` — disables animated spinner in parallel mode
- `--result-json path` — writes detailed JSON report of all commands and results
- `--fail-fast` — stops after first environment failure

Env var: `NO_COLOR=1` also disables colors. The spinner is auto-disabled when `CI=true`.

## Django (manage.py)

```bash
# Run tests (minimal output)
python manage.py test --no-color --verbosity 0 --no-input

# Run tests (with failure details)
python manage.py test --no-color --verbosity 1 --no-input --failfast

# System check
python manage.py check --no-color --verbosity 0

# Migrations
python manage.py migrate --no-color --verbosity 0 --no-input
```

- `--no-color` — disables ANSI colors (all management commands)
- `--verbosity 0` / `-v 0` — minimal output (0=minimal, 1=normal, 2=verbose, 3=very verbose)
- `--no-input` / `--noinput` — disables interactive prompts (prevents hanging)
- `--failfast` — stops test suite after first failure
- `--buffer` / `-b` — discards output from passing tests (only shows output on failure)
- `--parallel [N]` — runs tests in parallel (`auto` for one per core)

Django wraps Python's `unittest` — it does not support pytest-style `--tb` or `--no-header` flags. For richer test output control, use pytest with `pytest-django` instead.

## Flask

```bash
# Run development server (no auto-reload, no debugger)
flask run --no-debugger --no-reload
```

- `--no-debugger` — disables the interactive debugger (prevents agents from hitting debugger prompts)
- `--no-reload` — disables auto-reloader (prevents restarts during agent workflows)

Flask's CLI is thin — most AI-relevant configuration happens through environment variables: `FLASK_DEBUG=0` disables debug mode entirely. Flask has no `--no-color` or `--quiet` flags — use `NO_COLOR=1` env var to suppress ANSI output from Werkzeug.

## AGENTS.md / CLAUDE.md Template

Copy this into your project's AGENTS.md or CLAUDE.md:

```markdown
## Running Tools

- Install: `pip install -q --no-color --disable-pip-version-check --no-input`
- Install (uv): `uv pip install -q --color never`
- Tests: `pytest --no-header --tb=short -q --color=no`
- Tests (single): `pytest --no-header --tb=short -q --color=no tests/test_file.py::test_name`
- Tests (Django): `python manage.py test --no-color --verbosity 0 --no-input`
- Type check (mypy): `mypy src/ --no-color-output --no-error-summary --no-pretty --show-column-numbers --show-error-codes`
- Type check (pyright): `pyright src/`
- Lint (ruff): `ruff check src/ --output-format=concise`
- Lint fix (ruff): `ruff check src/ --fix --unsafe-fixes`
- Format check (ruff): `ruff format --check -q src/`
- Format fix (ruff): `ruff format src/`
- Format check (black): `black --check --diff src/`
- Format fix (black): `black src/`
- Lint (flake8): `flake8 src/`
- Lint (pylint): `pylint src/ --score=no`
- Import sort check: `isort --check --diff src/`
- Import sort fix: `isort src/`
- Security scan: `bandit -r src/ -f custom --msg-template "{abspath}:{line}: {test_id}: {msg} [{severity}:{confidence}]"`
- Coverage: `coverage run -m pytest tests/ && coverage report --format=markdown --skip-covered --skip-empty -m`
- Tests (tox): `tox --colored no -q`
```
