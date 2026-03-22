# Kotlin

## Quick Reference

| Tool              | AI-Friendly Flags                                                   |
|-------------------|---------------------------------------------------------------------|
| ./gradlew (build) | `--console=plain` (add `-q` for minimal output)                     |
| ./gradlew test    | `--console=plain`                                                   |
| kotlinc           | No flags needed (no colors, no progress bars)                       |
| ktlint            | `--relative --log-level=none`                                       |
| ktlint (fix)      | `-F --relative --log-level=none`                                    |
| detekt            | No flags needed (default output is one-line-per-finding, no colors) |
| spotless (Gradle) | `--console=plain`                                                   |

## ./gradlew

```bash
# Build — clean output
./gradlew build --console=plain

# Errors only
./gradlew build --console=plain -q
```

- `--console=plain` — disables all colors, progress bars, and rich status output
- `-q` / `--quiet` — suppresses most output (errors and explicit stdout from build scripts still show)
- `-w` / `--warn` — shows warnings and errors only
- `--no-daemon` — runs without the Gradle daemon (avoids stale daemon issues in CI)
- `--warning-mode=summary` — shows deprecation warning summary instead of individual warnings

With `--console=auto` (default), Gradle auto-detects TTY via `isatty()` and uses plain text in non-TTY environments. `--console=plain` makes this explicit.

Gradle does not respect `NO_COLOR=1`. Use `--console=plain` instead.

Persistent config via `gradle.properties`:

```properties
org.gradle.console=plain
org.gradle.daemon=false
```

### ./gradlew test

```bash
./gradlew test --console=plain

# Single test class
./gradlew test --console=plain --tests "com.example.MyTest"

# Single test method
./gradlew test --console=plain --tests "com.example.MyTest.myMethod"
```

Test results are saved as JUnit XML in `build/test-results/test/` automatically. To control console test output, configure `testLogging` in `build.gradle.kts`:

```kotlin
tasks.test {
    testLogging {
        events("failed")
        exceptionFormat = org.gradle.api.tasks.testing.logging.TestExceptionFormat.FULL
    }
}
```

- `events("failed")` — stream individual test failure events to console (default shows no individual test events — only the final pass/fail count)
- `exceptionFormat = FULL` — show full stack traces for failures

## kotlinc

```bash
kotlinc -Werror src/main/kotlin/
```

- `-Werror` — treats warnings as errors (exits 1 on any warning)
- `-nowarn` — suppresses all warnings
- `-Wextra` — enables extra checkers (Kotlin 2.0+)
- `-verbose` — enables verbose logging output
- `-progressive` — enables progressive mode where deprecations take effect immediately

kotlinc outputs errors and warnings to stderr in `file:line:col: severity: message` format, followed by the source line and a `^` caret pointer (typically 3 lines per diagnostic). No color or progress flags exist because kotlinc produces no colors or progress bars.

kotlinc is rarely invoked directly — Gradle handles compilation. These flags are configured in `build.gradle.kts`:

```kotlin
tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile> {
    compilerOptions {
        allWarningsAsErrors.set(true)
    }
}
```

## ktlint

```bash
# Check
ktlint --relative --log-level=none "src/**/*.kt"

# Single file
ktlint --relative --log-level=none src/main/kotlin/MyFile.kt

# Fix
ktlint -F --relative --log-level=none "src/**/*.kt"
```

- `--relative` — prints file paths relative to the working directory instead of absolute paths
- `--log-level=none` — suppresses informational and warning log messages printed to stderr
- `-F` / `--format` — auto-corrects fixable violations in place
- `--color` — opt-in flag to enable ANSI color codes (colors are off by default)
- `--limit=<n>` — caps the number of errors shown
- `--reporter=<name>` — selects output format

Default output (plain reporter) is already `file:line:col: message (rule)` — one violation per line, no colors unless `--color` is explicitly passed.

ktlint always prints a summary footer after violations listing error counts by rule. This footer cannot be suppressed.

Other reporters: `plain?group_by_file`, `plain-summary` (counts only, no individual violations), `json`, `sarif`, `checkstyle`, `html`.

Via Gradle (ktlint-gradle plugin):

```bash
./gradlew ktlintCheck --console=plain
./gradlew ktlintFormat --console=plain
```

## detekt

```bash
# Check
detekt --input src/

# Single file
detekt --input src/main/kotlin/MyFile.kt
```

- `--auto-correct` / `-ac` — auto-corrects violations where supported (requires the `formatting` ruleset plugin)
- `--report` / `-r` — generates report files: `-r txt:report.txt -r xml:report.xml`
- `--base-path` / `-bp` — sets base directory for relative paths in report files (does not affect console output)
- `--parallel` — enables parallel analysis (beneficial for codebases over ~2000 lines)
- `--all-rules` — activates all available rules including unstable ones
- `--max-issues=<n>` — exits 0 if found issues count does not exceed the threshold
- `--baseline` / `-b` — compares against a baseline file, only reporting new findings
- `--build-upon-default-config` — uses default config as base, allowing overrides

Default output is already `file:line:col: message [RuleName]` — one finding per line, no colors, no progress bars. A summary line ("Analysis failed with N weighted issues.") is always appended.

Report formats: `txt`, `xml`, `html`, `md`, `sarif`. Multiple reports can be generated simultaneously: `-r txt:detekt.txt -r sarif:detekt.sarif`.

Via Gradle (detekt-gradle plugin):

```bash
./gradlew detekt --console=plain
```

## spotless (Gradle plugin)

```bash
# Check formatting
./gradlew spotlessCheck --console=plain

# Fix formatting
./gradlew spotlessApply --console=plain
```

Noise comes from Gradle, not Spotless itself — use `--console=plain` to suppress it.

## AGENTS.md / CLAUDE.md Template

Copy this into your project's AGENTS.md or CLAUDE.md:

```markdown
## Running Tools

- Build: `./gradlew build --console=plain`
- Tests: `./gradlew test --console=plain`
- Tests (single): `./gradlew test --console=plain --tests "com.example.MyTest.myMethod"`
- Lint (ktlint): `./gradlew ktlintCheck --console=plain`
- Lint fix (ktlint): `./gradlew ktlintFormat --console=plain`
- Static analysis (detekt): `./gradlew detekt --console=plain`
- Format check (Spotless): `./gradlew spotlessCheck --console=plain`
- Format fix (Spotless): `./gradlew spotlessApply --console=plain`
```
