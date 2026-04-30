# finance-cli-skill

A [Claude](https://claude.ai/code) skill that drives
[finance-cli](https://github.com/kelvin6365/finance-cli) — a local,
offline, single-binary terminal finance tracker.

The skill teaches Claude to:

- Self-discover the command surface via `finance schema --json` on first run
- Always emit machine-parseable `--json` output and read the stable envelope
- Use `--dry-run` before destructive operations (`delete`, `loan pay`, …)
- Use `--idempotency-key` on retry-prone mutations (24h dedupe window)
- Translate intents like *"can I afford 5000 by may 31?"* or *"should I push freelance?"* into the right command(s)

## Install

Requires the `finance` CLI on your `PATH`. If you don't have it yet:

```bash
curl -fsSL https://raw.githubusercontent.com/kelvin6365/finance-cli/main/install.sh | bash
# or, with Bun:
bun add -g github:kelvin6365/finance-cli
```

Then install this skill:

```bash
git clone https://github.com/kelvin6365/finance-cli-skill ~/.claude/skills/finance-cli
```

Or download the packaged `.skill` from the
[Releases page](../../releases) and drop it in `~/.claude/skills/`.

## Try it

Open a Claude session and ask in natural language:

- "log $80 lunch"
- "can I afford 5000 by may 31?"
- "should I push freelance this month?"
- "what if I throw $5,000 extra at debt avalanche-style?"
- "what changed since 2026-04-01?"

## Layout

```
finance-cli-skill/
├── SKILL.md                 core skill body (triggers + intent→command map)
└── references/
    ├── envelope.md          JSON envelope shape, error codes, idempotency
    └── recipes.md           multi-step workflow patterns
```

## License

MIT — see [LICENSE](LICENSE).
