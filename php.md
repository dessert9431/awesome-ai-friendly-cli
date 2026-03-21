# PHP

## Quick Reference

| Tool                            | AI-Friendly Flags                                                                       |
|---------------------------------|-----------------------------------------------------------------------------------------|
| PHPUnit                         | `--no-progress --colors=never --display-errors --display-warnings`                      |
| Pest                            | `--compact --colors=never`                                                              |
| ParaTest                        | `--no-progress --colors=never --display-errors --display-warnings`                      |
| PHPStan                         | `--no-progress --no-ansi --error-format=raw`                                            |
| Psalm                           | `--no-progress --monochrome --show-snippet=false --no-suggestions --output-format=text` |
| phpcs                           | `--report=emacs -q --no-colors`                                                         |
| phpcbf                          | `-q --no-colors`                                                                        |
| PHP-CS-Fixer                    | `--show-progress=none --no-ansi -n`                                                     |
| Pint                            | No extra flags needed — auto-outputs JSON in non-TTY                                    |
| Rector                          | `--no-progress-bar --no-ansi`                                                           |
| Composer install/require/update | `-q` (or `--no-progress --no-ansi -n` if you need output)                               |

## PHPUnit

```bash
vendor/bin/phpunit --no-progress --colors=never --display-errors --display-warnings
```

- `--no-progress` — removes dot progress indicator (`.FE...`)
- `--colors=never` — disables ANSI color codes
- `--display-errors` — shows PHP errors triggered during tests (hidden by default in PHPUnit 10+)
- `--display-warnings` — shows PHP warnings triggered during tests

The banner (version, runtime, config path) and footer (time, memory) are always printed — no flag to suppress them. The savings come from eliminating progress dots on large suites.

Single test: `vendor/bin/phpunit --no-progress --colors=never --filter=MethodName`

## Pest

```bash
./vendor/bin/pest --compact --colors=never
```

- `--compact` — condenses output to dot progress (`.` pass, `⨯` fail) instead of listing each test name
- `--colors=never` — disables colors

Pest always shows code snippets for failures — there's no `--no-snippet` flag. `--compact` only affects the progress display.

[//]: # (> **Note:** `php artisan test --compact` does not reliably pass through to Pest. Use `./vendor/bin/pest` directly.)

## ParaTest

```bash
./vendor/bin/paratest --no-progress --colors=never --display-errors --display-warnings
```

Same flags as PHPUnit — ParaTest passes them through.

Via Laravel: `php artisan test --parallel --no-progress --colors=never --display-errors --display-warnings`

## PHPStan

```bash
vendor/bin/phpstan analyse --no-progress --no-ansi --error-format=raw
```

- `--error-format=raw` — one error per line: `file:line:message`
- `--no-progress` — disables progress bar
- `--no-ansi` — disables colors

Other formats: `table` (default, noisy), `json`, `checkstyle`, `github`, `gitlab`, `junit`.

The `raw` format is the most token-efficient — one line per error. Note: PHPStan prints a `Note: Using configuration file...` line to stderr when auto-discovering a config file.

## Psalm

```bash
vendor/bin/psalm --no-progress --monochrome --show-snippet=false --no-suggestions --output-format=text
```

- `--output-format=text` — one error per line: `file:line:col:severity - IssueType: message`
- `--no-progress` — disables progress indicator
- `--monochrome` / `-m` — disables colors
- `--show-snippet=false` — hides code snippets (saves tokens)
- `--no-suggestions` — hides fix suggestions

[//]: # (> **Psalm 7+:** Use `--output-format=compact` instead of `text` — it becomes a true one-line-per-error format &#40;[PR #11750]&#40;https://github.com/vimeo/psalm/pull/11750&#41;&#41;. In Psalm 6 and below, `compact` misleadingly renders box-drawing tables.)

Other formats: `compact`, `json`, `by-issue-level`, `github`, `checkstyle`, `emacs`, `sarif`.

## PHP_CodeSniffer (phpcs)

```bash
vendor/bin/phpcs --report=emacs -q --no-colors
```

- `--report=emacs` — one error per line: `file:line:col: severity - message`
- `-q` — disables progress dots and verbose output
- `--no-colors` — disables colors

Add `-s` to include sniff names (e.g. `PSR12.Files.FileHeader.SpacingAfterBlock`) — useful for agents to look up rules, but doubles line length.

Other formats: `full` (default, box borders), `json`, `summary`, `checkstyle`, `csv`, `diff`, `junit`.

## PHP_CodeSniffer auto-fixer (phpcbf)

```bash
vendor/bin/phpcbf -q --no-colors
```

- `-q` — disables progress dots (`F 1 / 1 (100%)`) and time/memory footer
- `--no-colors` — disables colors (already the default, but explicit is safer)

The result summary table is always printed — no flag to suppress it:

```
PHPCBF RESULT SUMMARY
----------------------------------------------------------------------
FILE                                                  FIXED  REMAINING
----------------------------------------------------------------------
/path/to/file.php                                     10     0
----------------------------------------------------------------------
A TOTAL OF 10 ERRORS WERE FIXED IN 1 FILE
----------------------------------------------------------------------
```

Useful filters: `--filter=GitModified` (only modified files), `--filter=GitStaged` (only staged files).

## PHP-CS-Fixer

```bash
# Check what would change
vendor/bin/php-cs-fixer check --show-progress=none --no-ansi -n --diff

# Fix
vendor/bin/php-cs-fixer fix --show-progress=none --no-ansi -n
```

- `--show-progress=none` — disables progress bar
- `--no-ansi` — disables colors
- `-n` — no interactive prompts
- `--diff` — shows unified diff of changes

The banner (version, runtime, config, cache info) is always printed — there's no flag to suppress it. `-q` suppresses ALL output including errors, so don't use it.

## Pint (Laravel)

```bash
./vendor/bin/pint --test
```

Pint auto-detects non-TTY environments and switches to compact JSON:

```json
{"result":"pass","files":[]}
{"result":"fail","files":[{"path":"file.php","fixers":["braces_position"]}]}
{"result":"fixed","files":[{"path":"file.php","fixers":["braces_position"]}]}
```

No extra flags needed.

Other useful flags: `--dirty` (only uncommitted files), `--diff=main` (only changed since branch).

## Rector

```bash
vendor/bin/rector process --no-progress-bar --no-ansi
```

- `--no-progress-bar` — disables progress bar
- `--no-ansi` — disables colors

Rector is best used as a fixer — let it apply changes directly rather than dry-running. Its console output shows unified diffs and applied rule names.

For structured output: `--output-format=json`

## Composer

```bash
# Minimal output (warnings and errors only)
composer install -q

# Verbose but clean
composer install --no-progress --no-ansi -n

# Security audit
composer audit --format=plain --no-ansi

# Check outdated (direct deps, structured)
composer outdated --direct --format=json --no-ansi
```

- `-q` — only warnings and errors (suppresses "182 packages looking for funding" noise)
- `--no-progress` — disables download progress bars
- `-n` — no interactive prompts

Env var equivalents: `COMPOSER_NO_INTERACTION=1` (same as `-n`), `NO_COLOR=1` (disables ANSI).

## AGENTS.md / CLAUDE.md Template

Copy this into your project's AGENTS.md or CLAUDE.md:

```markdown
## Running Tools

- Tests: `vendor/bin/phpunit --no-progress --colors=never --display-errors --display-warnings`
- Tests (with a filter): `vendor/bin/phpunit --no-progress --colors=never --filter=MethodName`
- Tests (Pest): `vendor/bin/pest --compact --colors=never`
- Static analysis (PHPStan): `vendor/bin/phpstan analyse --no-progress --no-ansi --error-format=raw`
- Static analysis (Psalm): `vendor/bin/psalm --no-progress --monochrome --show-snippet=false --no-suggestions --output-format=text`
- Code style check: `vendor/bin/phpcs --report=emacs -q --no-colors`
- Code style fix (phpcs): `vendor/bin/phpcbf -q --no-colors`
- Code style fix (php-cs-fixer): `vendor/bin/php-cs-fixer fix --show-progress=none --no-ansi -n`
- Refactor: `vendor/bin/rector process --no-progress-bar --no-ansi`
```
