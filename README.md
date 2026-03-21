# awesome-ai-friendly-cli

Flags and configs to silence CLI noise for AI agents. Save 60-90% of wasted tokens.

AI agents don't need progress bars, color codes, banners, or interactive prompts.
Most CLI tools have flags to turn these off — but they're buried in docs and scattered across ecosystems.

This repo collects them in one place.

## Ecosystems

- [Node.js](node.md)
- [Python](python.md)
- [PHP](php.md) — php-cs-fixer, phpcs, phpstan, psalm, phpunit, pest
- [Rust](rust.md)
- [Go](go.md)
- [Ruby](ruby.md)
- [Java](java.md)
- [.NET](dotnet.md)

## Common patterns

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

### [rtk](https://github.com/rtk-ai/rtk) — CLI proxy that compresses output automatically

[rtk](https://github.com/rtk-ai/rtk) is a single Rust binary that sits between your AI agent and the shell. It intercepts commands, runs them, and compresses the output before it reaches the LLM context window — smart filtering, grouping similar items, truncating boilerplate, and deduplicating repeated lines.

It hooks into Claude Code, Gemini CLI, and OpenCode automatically (auto-rewrites `git status` → `rtk git status`). ~10ms overhead per command.

Typical savings: 60-90% token reduction across git, test, lint, and build commands.

rtk and per-tool flags are complementary — flags control what the tool *produces*, rtk compresses what the agent *receives*. Use both for maximum savings.

## Why this matters

A typical `npm install` output burns ~2,000 tokens. With `--no-progress --no-fund --no-audit`, it drops to ~200.

Multiply that across every tool call in an agentic workflow, and the savings are massive.

## Contributing

PRs welcome. Add flags, fix errors, cover new tools. Keep entries minimal and tested.

## License

[CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/) — public domain.
