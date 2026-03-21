# Contribution Guidelines

Thanks for taking the time to contribute! This list is useful because people like you keep it accurate and up-to-date.

## What belongs here

CLI flags and environment variables that reduce noise when tools are invoked by AI agents. Specifically:

- Flags that disable progress bars, colors, banners, or interactive prompts.
- Flags that switch output to machine-readable formats (JSON, one-line-per-error, etc.).
- Environment variables that achieve the same (e.g., `NO_COLOR=1`, `CI=true`).
- Non-obvious behavior worth documenting (e.g., "without `run`, vitest enters watch mode").

## What does NOT belong

- Tool reviews or opinions ("this tool is better than X").
- Flags unrelated to output noise (e.g., `--threads=4`).
- Tools that are already AI-agent-specific (they don't need quieting).
- Tools with low adoption

## Format

Each ecosystem file follows the same structure:

1. **Quick Reference table** at the top — one row per tool, most useful flags.
2. **One section per tool** with a code block showing the recommended command, followed by a bullet list explaining each flag.
3. Keep descriptions factual and short. No marketing language.

Example entry:

```markdown
## toolname

\`\`\`bash
toolname check --no-progress --no-color
\`\`\`

- `--no-progress` — disables progress bar
- `--no-color` — disables ANSI color codes
```

## Pull request checklist

- [ ] I verified the flags work with the current stable version of the tool.
- [ ] I added the tool to the Quick Reference table at the top of the ecosystem file.
- [ ] Descriptions are factual (no "best", "awesome", "amazing").
- [ ] Each description ends without a period (matching existing style in tool sections).
- [ ] I searched existing entries to avoid duplicates.

## Adding a new ecosystem

If you want to add a new ecosystem (e.g., Elixir, Swift):

1. Create a new `{ecosystem}.md` file following the format above.
2. Add it to the Ecosystems list in `README.md` with a short description ending with a period.

## Other contributions

- Fix outdated flags (tools change between versions — corrections are very welcome).
- Add missing tools to existing ecosystem files.
- Improve descriptions for clarity.
- Report inaccuracies by opening an issue.
