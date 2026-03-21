# Awesome AI-Friendly CLI

> Flags and configs to silence CLI noise for AI agents. Save 60-90% of wasted tokens.

AI agents don't need progress bars, color codes, banners, or interactive prompts. Most CLI tools have flags to turn these off — but they're buried in docs and scattered across ecosystems. This repo collects them in one place.

## Contents

- [Ecosystems](#ecosystems)
- [Common Patterns](#common-patterns)
- [Tools](#tools)

## Ecosystems

- [Node.js](node.md) - npm, npx, yarn, pnpm, eslint, prettier, biome, jest, vitest, mocha, tsc, playwright, cypress
- [Python](python.md) - WIP
- [PHP](php.md) - php-cs-fixer, phpcs, phpstan, psalm, phpunit, pest
- [Rust](rust.md) - WIP
- [Go](go.md) - WIP
- [Ruby](ruby.md) - WIP
- [Java](java.md) - WIP
- [.NET](dotnet.md) - WIP

## Common Patterns

Most CLI tools respect some combination of:

| Flag / Env          | What it does                                                             |
|---------------------|--------------------------------------------------------------------------|
| `--no-progress`     | Disables progress bars                                                   |
| `--no-color`        | Disables ANSI color codes                                                |
| `--quiet` / `-q`    | Minimal output                                                           |
| `--json`            | Machine-readable output                                                  |
| `--non-interactive` | Disables prompts and confirmations                                       |
| `NO_COLOR=1`        | Standard env var to disable color ([no-color.org](https://no-color.org)) |
| `CI=true`           | Many tools auto-quiet when they detect CI                                |

## Tools

- [rtk](https://github.com/rtk-ai/rtk) - CLI proxy that sits between AI agents and the shell, compressing output before it reaches the LLM context window. Works with Claude Code, Gemini CLI, and OpenCode.

## Contributing

Contributions welcome! Read the [contribution guidelines](CONTRIBUTING.md) first.
