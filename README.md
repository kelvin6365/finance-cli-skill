# finance-cli-skill

[![skills.sh](https://img.shields.io/badge/skills.sh-installable-black)](https://skills.sh/docs)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

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

### 1. The CLI itself

This skill drives `finance` — install the binary first if you don't have it:

```bash
curl -fsSL https://raw.githubusercontent.com/kelvin6365/finance-cli/main/install.sh | bash
# or, with Bun:
bun add -g github:kelvin6365/finance-cli
```

(The skill will guide you through this on first use too — it detects when
the binary is missing and walks you through install + `finance init`.)

### 2. The skill

**Recommended — via [skills.sh](https://skills.sh/docs):**

```bash
npx skills add kelvin6365/finance-cli-skill
```

The `skills` CLI (open source at
[vercel-labs/skills](https://github.com/vercel-labs/skills)) detects your
agent (Claude Code, Cursor, etc.) and drops the skill into the right
directory automatically. No manual paths to manage.

**Manual install (git):**

```bash
git clone https://github.com/kelvin6365/finance-cli-skill ~/.claude/skills/finance-cli
```

**Manual install (packaged):** download the `.skill` from the
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
