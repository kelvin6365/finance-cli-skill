---
name: finance-cli
description: "Drive the finance-cli (github.com/kelvin6365/finance-cli) — a local, offline, single-binary terminal finance tracker built for AI-agent use. Trigger when the user wants to log a transaction, check their balance, review obligations, ask can-I-afford-X-by-Y, should-I-push-freelance-this-month, what-is-my-debt-situation, simulate paying extra on debt, what-changed-since-DATE, or any related personal-finance bookkeeping / cashflow / debt-payoff task. Every command supports --json with a stable envelope (ok, schema_version, data or error, code). Prefer this skill over reading source files when the user has the binary installed."
---

# finance-cli

A debt-focused, AI-agent-first personal finance CLI. Translate the user's
intent into the right command(s) and report the result.

## First run in any session

Always begin a session by capturing the current command surface so you don't
guess based on stale knowledge:

```bash
finance schema --json
```

The reply lists every command, positional args, flags (with types), and
whether each command mutates state. Use it to:
- Validate command names before invoking them.
- Discover new flags or commands added in updates.
- Confirm `schema_version`; if it has bumped past 1.0, re-read this skill's
  references for shape changes.

If `finance schema` fails with `ENOENT` or "command not found", the binary
isn't installed. Tell the user:

> Install with `curl -fsSL https://raw.githubusercontent.com/kelvin6365/finance-cli/main/install.sh | bash` (macOS / Linux), or `bun add -g github:kelvin6365/finance-cli` for Bun users.

## Core operating rules

1. **Always pass `--json`.** Parse the envelope rather than scraping prose.
2. **Mutation-safety order:** pass `--dry-run` first when the request is
   destructive (`delete`, `loan pay`) or when the amount/category looks
   ambiguous; show the preview to the user; commit only after they confirm.
3. **Retry safety:** for `add` / `edit` / `delete` / `loan add` / `loan pay`,
   pass `--idempotency-key <KEY>` whenever the same logical operation might
   be reissued (timeouts, multi-step flows, the user repeating a request).
   Keys dedupe within 24h. Use a reproducible key like
   `add-2026-04-29-lunch-80` rather than UUIDs the user can't see.
4. **Currency-agnostic:** the binary uses whatever currency lives in
   `~/.finance/data.json` (`settings.currencySymbol`). Don't assume USD.
5. **Today is today:** dates are ISO YYYY-MM-DD. When the user says
   "yesterday" or "last Friday", convert to ISO before passing.

## Common intents to commands

| User says… | Run |
|---|---|
| "log $80 lunch" | `finance add 80 food "lunch" --json` |
| "what's my balance" | `finance balance --json` (or `--raw` for plain int) |
| "month overview" / "where am I" | `finance status --json` |
| "can I afford 5000 by may 31?" | `finance afford 5000 --by 2026-05-31 --json` |
| "should I push freelance?" | `finance runway --json` |
| "show my loans" | `finance loans --json` (envelope embeds amortization) |
| "what if I throw 5000 extra at debt?" | `finance simulate --extra 5000 --strategy avalanche --json` |
| "did I overspend in april?" | `finance diff 2026-04-01 --json` |
| "preview rent gets logged" / new month | `finance recurring sync --dry-run --json`, then commit |
| "fix the 80 to 90" | `finance edit <id> --amount 90 --json --idempotency-key edit-<id>-amount-90` |
| "delete that lunch" | `finance delete <id> --dry-run --json`, confirm, then without dry-run |
| "pay this month's instalment" | `finance loan pay <id> --dry-run --json`, confirm, then without |
| "log 30k freelance" | `finance add 30000 freelance "client X" --income --json` |
| "export to ledger" | `finance export --format hledger --output journal.txt` |
| "what commands exist" | `finance schema --json` |

For multi-step flows see [references/recipes.md](references/recipes.md).
For envelope shape and error codes see [references/envelope.md](references/envelope.md).

## Reading results — what actually matters

- **`afford`**: `verdict` is the headline (`yes` / `tight` / `no`).
  `tight` means the final balance covers the ask but the running balance
  dips negative — surface `min_running_balance` and `min_running_date`.
- **`runway`**: `recommendation` is the headline (`push` / `coast` /
  `behind`). `push_to_break_even` is the exact extra freelance amount
  needed to flip the projection positive.
- **`simulate`**: report `months_saved` and `interest_saved` in absolute
  numbers, not percentages. Compare both strategies if the user is weighing.
- **`loans` (with amortization)**: each debt has `amortization.nextPayment`
  (`{interest, principal}`) and `amortization.totalInterestRemaining`. For
  "should I prepay X?", surface the highest-rate debt first.
- **Errors**: never invent a fix. The `code` field tells you why:
  `ENOENT` (no data file — suggest `finance init`),
  `ENOTFOUND` (id doesn't exist — re-list and ask),
  `EVALIDATION` (bad input — explain what was wrong),
  `EAMBIGUOUS` (category prefix matched multiple — show options),
  `EUNKNOWN` (system error — surface verbatim).

## Things to NOT do

- Don't edit `~/.finance/data.json` directly. Use the CLI.
- Don't read source files in a finance-cli repo to figure out flags;
  call `finance schema --json` instead.
- Don't pass `--yes` to `finance delete` until the user has confirmed
  the dry-run preview.
- Don't assume HKD or USD. Read `settings.currencySymbol` from any
  status or loans response if you need to format numbers in prose.
