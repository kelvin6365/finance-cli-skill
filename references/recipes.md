# Workflow recipes

Multi-step patterns Claude should follow for non-trivial requests.

## Recipe: log a transaction safely

1. If the category isn't certain, run `finance categories --json` and
   pick the closest match by name or id.
2. Build an idempotency key the user could replay later, e.g.
   `add-2026-04-29-lunch-80`.
3. Run with `--dry-run --json` first if the amount looks unusual or the
   user said "I think it was about…".
4. Commit: drop `--dry-run`, keep the `--idempotency-key`.

## Recipe: month-start sweep (recurring sync)

1. `finance recurring sync --dry-run --json` — preview pending entries.
2. Show the list to the user (date / amount / name) and ask if anything
   should be skipped.
3. If clean: `finance recurring sync --json` to commit. The command is
   idempotent on its own — re-running will create 0 new entries — so
   `--idempotency-key` is unnecessary here.

## Recipe: "should I push freelance this month?"

1. `finance runway --json` — read `recommendation` and `push_to_break_even`.
2. If `recommendation === "push"`, give the user the exact gap figure
   in their currency.
3. Optionally cross-check with `finance afford <amount> --by <month-end>
   --json` for a specific bill they're worried about.

## Recipe: "should I prepay loan X?"

1. `finance loans --json` — find the loan, note its `annualRate` and
   `amortization.totalInterestRemaining`.
2. `finance simulate --extra <amount> --strategy avalanche --json` and
   then `--strategy snowball --json`.
3. Compare `interest_saved` and `months_saved`. If avalanche wins by
   >2x interest, recommend it; if snowball wins on a "small balance
   gone" basis, mention the psychological vs financial tradeoff.

## Recipe: deletion / correction

1. Look up the transaction with `finance show <id> --json` or
   `finance list --json` filters.
2. `finance delete <id> --dry-run --json` (or `finance edit <id>
   --amount N --dry-run --json`) — show the preview.
3. Wait for explicit user confirmation.
4. Re-run without `--dry-run`, with `--idempotency-key delete-<id>` so
   accidental re-issuance is a no-op.

## Recipe: "what changed since <date>"

`finance diff <YYYY-MM-DD> --json` returns income, expense, net, and
top categories for the window. Surface the top 3 categories by absolute
net to summarise the period.

## Recipe: hand off to a plain-text-accounting tool

1. `finance export --format hledger --output ~/finance.journal --json`.
2. Tell the user where it landed (`data.path` in the envelope).
3. They can `hledger -f ~/finance.journal bal` from there.
