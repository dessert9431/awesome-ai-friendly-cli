# Java

## Quick Reference

| Tool                | AI-Friendly Flags                                                   |
|---------------------|---------------------------------------------------------------------|
| mvn (install/build) | `-B -ntp --color=never` (or `-q` for errors only)                   |
| mvn test            | `-B -ntp --color=never`                                             |
| ./gradlew (build)   | `--console=plain` (add `-q` for minimal output)                     |
| ./gradlew test      | `--console=plain`                                                   |
| javac               | `-Xlint:all` (warnings already go to stderr, no noise flags needed) |
| checkstyle          | No flags needed (default output is already one-line-per-violation)  |
| pmd check           | `--no-progress --format text`                                       |
| spotbugs            | `-textui -emacs`                                                    |
| google-java-format  | `--dry-run --set-exit-if-changed` (check) or `--replace` (fix)      |
| spotless (Gradle)   | `--console=plain`                                                   |
| spotless (Maven)    | `-B -ntp --color=never`                                             |

## mvn

```bash
# Install — clean output
mvn -B -ntp --color=never clean install

# Errors only
mvn -B -q -ntp --color=never clean install
```

- `-B` / `--batch-mode` — disables interactive prompts and colors
- `-ntp` / `--no-transfer-progress` — suppresses artifact download/upload progress bars
- `--color=never` — explicitly disables ANSI color codes
- `-q` / `--quiet` — shows errors only (suppresses all non-error output)
- `-e` / `--errors` — shows execution error messages including stack traces

`-B` disables colors automatically, but `--color=never` is explicit and safer. Maven does not respect `NO_COLOR=1` or `CI=true` — use CLI flags.

In non-TTY environments, Maven auto-disables colors via JAnsi `isatty()` detection. Transfer progress bars are still shown — `-ntp` is needed to suppress them.

Env var: `MAVEN_ARGS="-B -ntp"` (Maven 3.9.0+) sets default flags for all invocations.

### mvn test (Surefire)

```bash
mvn -B -ntp --color=never test

# Single test class
mvn -B -ntp --color=never test -Dtest=MyTest

# Single test method
mvn -B -ntp --color=never test -Dtest=MyTest#myMethod
```

- `-Dtest=ClassName` — runs a specific test class
- `-Dtest=ClassName#method` — runs a specific test method
- `-Dmaven.test.redirectTestOutputToFile=true` — redirects test stdout/stderr to `target/surefire-reports/testName-output.txt` instead of console
- `-Dmaven.test.failure.ignore=true` — continues build even if tests fail

Surefire generates machine-readable XML reports in `target/surefire-reports/TEST-*.xml` automatically — no flags needed.

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

With `--console=auto` (default), Gradle auto-detects TTY and uses plain text in non-TTY environments. `--console=plain` makes this explicit.

Gradle does not respect `NO_COLOR=1`. Use `--console=plain` instead.

Persistent config via `gradle.properties`:

```properties
org.gradle.console=plain
org.gradle.daemon=false
```

### ./gradlew test

```bash
./gradlew test --console=plain
```

Test results are saved as JUnit XML in `build/test-results/test/` automatically. To control console test output, configure `testLogging` in `build.gradle`:

```groovy
test {
    testLogging {
        events "failed"
        exceptionFormat "full"
    }
}
```

- `events "failed"` — stream individual test failure events to console (default shows only a summary at the end)
- `exceptionFormat "full"` — show full stack traces for failures

Single test: `./gradlew test --console=plain --tests "com.example.MyTest.myMethod"`

## javac

```bash
javac -Xlint:all src/*.java
```

- `-Xlint:all` — enables all compiler warnings
- `-Xlint:none` / `-nowarn` — disables all warnings
- `-Xlint:-deprecation` — disables a specific warning category (use `-` prefix)

javac outputs errors and warnings to stderr in `filename:line: error: message` format — already one-line-per-error with no decorative noise. No color or progress flags exist because javac produces no colors or progress bars.

Common warning categories: `cast`, `deprecation`, `divzero`, `empty`, `fallthrough`, `finally`, `overrides`, `path`, `rawtypes`, `serial`, `static`, `try`, `unchecked`, `varargs`, `removal`. Run `javac --help-lint` for the full list (count varies by Java version).

javac is rarely invoked directly — Maven and Gradle handle compilation. These flags are configured in build tool settings.

## checkstyle

```bash
java -jar checkstyle.jar -c /google_checks.xml src/
```

- `-f plain` — plain text output (default): `[SEVERITY] filepath:line:col: message [CheckName]`
- `-f xml` — XML format
- `-f sarif` — SARIF format (for tool integrations)
- `-o <file>` — writes output to file instead of stdout

Checkstyle has no progress bar and no color output — the default plain format is already token-efficient with one violation per line.

Via Maven:

```bash
mvn -B -ntp --color=never checkstyle:check
```

Via Gradle:

```bash
./gradlew checkstyleMain --console=plain
```

Reports are written to `build/reports/checkstyle/` (Gradle) or `target/checkstyle-result.xml` (Maven).

## pmd

```bash
pmd check -d src/ -R rulesets/java/quickstart.xml --no-progress --format text
```

- `--no-progress` — disables live analysis progress bar (enabled by default)
- `--format text` — one violation per line (default)
- `--format json` — structured JSON output
- `--format csv` — CSV output
- `--format sarif` — SARIF format
- `--format codeclimate` — Code Climate JSON
- `--report-file <path>` — writes report to file instead of stdout
- `--no-fail-on-violation` — exits 0 even when violations are found

The `pmd check` subcommand requires PMD 7+. PMD 6.x uses `pmd -d src/ -R ...` without the `check` subcommand.

The default `text` format is already compact. Use `--no-progress` to suppress the progress bar — this is the main noise source. Additional formats include `xml`, `html`, and `summaryhtml`.

Via Maven:

```bash
mvn -B -ntp --color=never pmd:check
```

Via Gradle:

```bash
./gradlew pmdMain --console=plain
```

## spotbugs

```bash
spotbugs -textui -emacs target/classes/
```

- `-textui` — runs in text UI mode (command-line)
- `-emacs` — Emacs-compatible one-line-per-bug format
- `-xml=bugs.xml` — XML format (use `-xml:withMessages` to include human-readable messages)
- `-html=bugs.html` — HTML format
- `-sarif=bugs.sarif` — SARIF format
- `-quiet` — suppresses SpotBugs' own progress and informational messages during analysis
- `-effort:min` — fastest analysis (fewer checks)
- `-effort:max` — most thorough analysis

Multiple output formats can be specified simultaneously: `-xml=bugs.xml -html=bugs.html`.

SpotBugs is typically run through Maven or Gradle rather than standalone:

```bash
# Maven
mvn -B -ntp --color=never spotbugs:check

# Gradle
./gradlew spotbugsMain --console=plain
```

## google-java-format

```bash
# Check which files need formatting
google-java-format --dry-run --set-exit-if-changed src/**/*.java

# Fix formatting
google-java-format --replace src/**/*.java
```

- `--dry-run` / `-n` — prints file paths that would change without modifying them
- `--set-exit-if-changed` — returns exit code 1 if files need formatting
- `--replace` / `-i` — formats files in-place
- `--aosp` / `-a` — uses AOSP style (4-space indent) instead of Google Style (2-space)
- `--fix-imports-only` — only fix import order and remove unused imports
- `--skip-javadoc-formatting` — skip javadoc reformatting
- `--skip-sorting-imports` — skip import sorting
- `--skip-removing-unused-imports` — skip unused import removal

google-java-format produces no progress bars, colors, or banners. Output is already clean.

## spotless (Gradle / Maven plugin)

Spotless has no standalone CLI — it runs through Gradle or Maven.

```bash
# Gradle — check formatting
./gradlew spotlessCheck --console=plain

# Gradle — fix formatting
./gradlew spotlessApply --console=plain

# Maven — check formatting
mvn -B -ntp --color=never spotless:check

# Maven — fix formatting
mvn -B -ntp --color=never spotless:apply
```

Noise comes from the build tool, not Spotless itself — use `--console=plain` (Gradle) or `-B -ntp --color=never` (Maven) to suppress it.

## AGENTS.md / CLAUDE.md Template

Copy this into your project's AGENTS.md or CLAUDE.md:

### Maven projects

```markdown
## Running Tools

- Install: `mvn -B -ntp --color=never clean install`
- Build: `mvn -B -ntp --color=never compile`
- Tests: `mvn -B -ntp --color=never test`
- Tests (single): `mvn -B -ntp --color=never test -Dtest=MyTest#myMethod`
- Lint (Checkstyle): `mvn -B -ntp --color=never checkstyle:check`
- Static analysis (PMD): `mvn -B -ntp --color=never pmd:check`
- Static analysis (SpotBugs): `mvn -B -ntp --color=never spotbugs:check`
- Format check (Spotless): `mvn -B -ntp --color=never spotless:check`
- Format fix (Spotless): `mvn -B -ntp --color=never spotless:apply`
```

### Gradle projects

```markdown
## Running Tools

- Build: `./gradlew build --console=plain`
- Tests: `./gradlew test --console=plain`
- Tests (single): `./gradlew test --console=plain --tests "com.example.MyTest.myMethod"`
- Lint (Checkstyle): `./gradlew checkstyleMain --console=plain`
- Static analysis (PMD): `./gradlew pmdMain --console=plain`
- Static analysis (SpotBugs): `./gradlew spotbugsMain --console=plain`
- Format check (Spotless): `./gradlew spotlessCheck --console=plain`
- Format fix (Spotless): `./gradlew spotlessApply --console=plain`
```
