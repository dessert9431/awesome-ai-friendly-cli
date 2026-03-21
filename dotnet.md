# .NET

## Quick Reference

| Tool           | AI-Friendly Flags                                           |
|----------------|-------------------------------------------------------------|
| dotnet build   | `--nologo -v:q --tl:off -clp:NoSummary`                     |
| dotnet test    | `--nologo -v:q --tl:off --logger "console;verbosity=quiet"` |
| dotnet publish | `--nologo -v:q --tl:off -clp:NoSummary`                     |
| dotnet pack    | `--nologo -v:q --tl:off -clp:NoSummary`                     |
| dotnet restore | `-v:q --tl:off`                                             |
| dotnet format  | `--verbosity quiet --verify-no-changes` (check)             |
| dotnet-ef      | `--no-color --prefix-output`                                |
| coverlet       | `--verbosity quiet --format cobertura`                      |

## Environment Variables

Set these globally for all .NET CLI commands:

| Variable                        | Effect                                          |
|---------------------------------|-------------------------------------------------|
| `DOTNET_NOLOGO=1`               | Suppresses welcome message and telemetry notice |
| `DOTNET_CLI_TELEMETRY_OPTOUT=1` | Opts out of telemetry data collection           |
| `MSBUILDTERMINALLOGGER=off`     | Disables Terminal Logger globally (see below)   |

`DOTNET_NOLOGO=1` replaces the older `DOTNET_SKIP_FIRST_TIME_EXPERIENCE=true` (no longer supported since .NET Core 3.0).

## Terminal Logger (`--tl`)

The Terminal Logger is an animated, interactive output mode for MSBuild-based commands. Available since .NET 8, **default since .NET 9**. It uses ANSI escape codes and cursor movement that produce garbage in captured output.

```bash
--tl:off    # Disable — use classic console logger (what agents want)
--tl:on     # Force on (even in non-TTY)
--tl:auto   # Auto-detect (default since .NET 9)
```

Supported by: `dotnet build`, `dotnet test`, `dotnet publish`, `dotnet pack`, `dotnet restore`.

In non-TTY environments, `--tl:auto` disables itself automatically. But `--tl:off` is safer to be explicit, especially since detection behavior varies across CI systems.

Env var equivalent: `MSBUILDTERMINALLOGGER=off`

## Verbosity Levels (`-v`)

All MSBuild-based commands (`build`, `test`, `publish`, `pack`, `restore`) support these levels. `dotnet format` also supports `--verbosity` with the same levels, though it is not MSBuild-based:

| Level        | Short  | Output                           |
|--------------|--------|----------------------------------|
| `quiet`      | `q`    | Errors only                      |
| `minimal`    | `m`    | Errors + warnings + minimal info |
| `normal`     | `n`    | Default MSBuild output           |
| `detailed`   | `d`    | Detailed build info              |
| `diagnostic` | `diag` | Full diagnostic output           |

Usage: `-v:q` or `--verbosity quiet`

## Console Logger Parameters (`-clp`)

Fine-grained control over MSBuild console output. Semicolon-separated. Supported by `build`, `test`, `publish`, `pack`.

| Parameter             | Effect                                   |
|-----------------------|------------------------------------------|
| `NoSummary`           | Hides error/warning summary at end       |
| `ErrorsOnly`          | Shows only errors                        |
| `WarningsOnly`        | Shows only warnings                      |
| `DisableConsoleColor` | Disables ANSI color codes                |
| `ForceNoAlign`        | Disables padding to console buffer width |

Example: `-clp:NoSummary;ForceNoAlign;DisableConsoleColor`

## dotnet build

```bash
dotnet build --nologo -v:q --tl:off --no-restore -clp:NoSummary;ForceNoAlign;DisableConsoleColor
```

- `--nologo` — suppresses startup banner and copyright
- `-v:q` — errors only
- `--tl:off` — disables Terminal Logger (default since .NET 9)
- `--no-restore` — skips implicit NuGet restore (use when already restored)
- `-clp:NoSummary` — hides the "Build succeeded" / "Build FAILED" summary
- `-clp:ForceNoAlign` — prevents padding to console width
- `-clp:DisableConsoleColor` — disables ANSI colors

With `-v:q` and `-clp:NoSummary`, a clean build produces zero output. Only errors are shown.

To suppress specific warnings: `-p:NoWarn=CS1591` (or `--property:NoWarn=CS1591`).

For structured build logs: `-bl` produces a binary log file for offline analysis with [MSBuild Structured Log Viewer](https://msbuildlog.com/).

## dotnet test

```bash
dotnet test --nologo -v:q --tl:off --no-restore --logger "console;verbosity=quiet"
```

- `--nologo` — suppresses Microsoft TestPlatform banner
- `-v:q` — minimal MSBuild output
- `--tl:off` — disables Terminal Logger
- `--no-restore` — skips implicit restore
- `--no-build` — skips building (use when already built, implies `--no-restore`)
- `--logger "console;verbosity=quiet"` — controls test runner output separately from MSBuild verbosity

Console logger verbosity levels: `quiet`, `minimal`, `normal`, `detailed`.

Single test: `dotnet test --filter "FullyQualifiedName~MyTestMethod"` or `dotnet test --filter "ClassName=MyTests"`

For structured output: `--logger "trx;logfilename=results.trx"` writes XML results. Multiple loggers can be combined:

```bash
dotnet test --logger "console;verbosity=quiet" --logger "trx;logfilename=results.trx"
```

The test results summary ("Passed! - Failed: 0, Passed: 42") always prints even at quiet verbosity — no flag suppresses it.

xUnit, NUnit, and MSTest all run via `dotnet test` — the flags above apply to all three. xUnit's `ITestOutputHelper` output only appears with `--logger "console;verbosity=detailed"`.

Other useful flags: `--blame` (identifies tests causing crashes), `--blame-hang-timeout 60s` (kills hanging tests).

## dotnet publish

```bash
dotnet publish --nologo -v:q --tl:off --no-restore -clp:NoSummary;ForceNoAlign;DisableConsoleColor
```

Same flags as `dotnet build`, plus:

- `--no-build` — skips building (implies `--no-restore`)

## dotnet pack

```bash
dotnet pack --nologo -v:q --tl:off --no-restore --no-build -clp:NoSummary;ForceNoAlign;DisableConsoleColor
```

Same flags as `dotnet build`, plus:

- `--no-build` — skips building (implies `--no-restore`)

## dotnet restore

```bash
dotnet restore -v:q --tl:off
```

- `-v:q` — errors only
- `--tl:off` — disables Terminal Logger
- `--ignore-failed-sources` — treats failed sources as warnings instead of errors

`dotnet restore` does not support `--nologo`.

Most workflows don't need explicit restore — `dotnet build` and `dotnet test` restore implicitly. Use `--no-restore` on those commands when you've already restored separately.

## dotnet format

```bash
# Check (non-zero exit if changes needed)
dotnet format --verbosity quiet --verify-no-changes --no-restore

# Fix
dotnet format --verbosity quiet --no-restore
```

- `--verbosity quiet` — minimal output
- `--verify-no-changes` — check mode, returns non-zero exit code if formatting changes are needed
- `--no-restore` — skips implicit restore
- `--report <PATH>` — produces a JSON report to the specified directory
- `--severity <info|warn|error>` — minimum diagnostic severity to fix
- `--include <paths>` — include only specified files/folders
- `--exclude <paths>` — exclude specified files/folders

Subcommands for targeted formatting: `dotnet format whitespace`, `dotnet format style`, `dotnet format analyzers` — all accept the same flags.

`dotnet format` does not support `--nologo`, `--tl`, or `-clp` — it is not MSBuild-based in the same way as build/test.

## dotnet-ef (Entity Framework Core)

```bash
# List migrations
dotnet ef migrations list --no-color --prefix-output --json --no-build

# Apply migrations
dotnet ef database update --no-color --prefix-output --no-build

# Scaffold from database
dotnet ef dbcontext scaffold --no-color --prefix-output --no-build
```

- `--no-color` — disables ANSI color codes
- `--prefix-output` — prefixes each line with log level (`info:`, `warn:`, `error:`), making output parseable
- `--json` — structured JSON output (accepted on all commands, but only produces meaningful JSON on `migrations list`, `dbcontext list`, `dbcontext info`)
- `--no-build` — skips building the project
- `--verbose` / `-v` — show verbose output

## coverlet

```bash
# Via dotnet test (simplest — using coverlet.collector package)
dotnet test --collect:"XPlat Code Coverage"

# Via MSBuild (using coverlet.msbuild package)
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=cobertura /p:CoverletOutput=./coverage/

# Via global tool
coverlet ./bin/test.dll --target dotnet --targetargs "test --no-build" \
  --output ./coverage/ --format cobertura --verbosity quiet
```

Global tool flags:

- `--format <fmt>` — output format: `json` (default), `lcov`, `opencover`, `cobertura`, `teamcity`
- `--output <path>` — output file/directory
- `--verbosity <level>` — `Quiet`, `Minimal`, `Normal` (default), `Detailed`
- `--threshold <N>` — fail if coverage below N%
- Multiple formats: `--format "cobertura,opencover"` (comma-separated)

The `--collect:"XPlat Code Coverage"` approach is the simplest — it outputs Cobertura XML to `TestResults/` with no extra configuration.

## AGENTS.md / CLAUDE.md Template

Copy this into your project's AGENTS.md or CLAUDE.md:

```markdown
## Running Tools

- Restore: `dotnet restore -v:q --tl:off`
- Build: `dotnet build --nologo -v:q --tl:off --no-restore`
- Tests: `dotnet test --nologo -v:q --tl:off --no-restore --logger "console;verbosity=quiet"`
- Tests (single): `dotnet test --nologo -v:q --tl:off --filter "FullyQualifiedName~TestMethodName"`
- Format check: `dotnet format --verbosity quiet --verify-no-changes --no-restore`
- Format fix: `dotnet format --verbosity quiet --no-restore`
- Publish: `dotnet publish --nologo -v:q --tl:off --no-restore`
- EF migrations: `dotnet ef migrations list --no-color --prefix-output --json --no-build`
```
