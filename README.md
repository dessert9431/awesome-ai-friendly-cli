# awesome-ai-friendly-cli

Flags and configs to silence CLI noise for AI agents. Save 60-90% of wasted tokens.

AI agents don't need progress bars, color codes, banners, or interactive prompts.
Most CLI tools have flags to turn these off — but they're buried in docs and scattered across ecosystems.

This repo collects them in one place.

## Ecosystems

- [Node.js](node.md) — npm, yarn, pnpm, npx
- [Python](python.md) — pip, poetry, uv, conda
- [Rust](rust.md) — cargo, rustup
- [Go](go.md) — go, go install
- [Ruby](ruby.md) — gem, bundler, rake
- [Java](java.md) — maven, gradle
- [.NET](dotnet.md) — dotnet, nuget
- [PHP](php.md) ✅ 

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

## Why this matters

A typical `npm install` output burns ~2,000 tokens. With `--no-progress --no-fund --no-audit`, it drops to ~200.

Multiply that across every tool call in an agentic workflow and the savings are massive.

## Contributing

PRs welcome. Add flags, fix errors, cover new tools. Keep entries minimal and tested.

## License

[CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/) — public domain.
