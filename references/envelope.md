# JSON envelope reference

Every command with `--json` emits exactly one line on stdout (success) or
stderr (error). The envelope shape is stable across `schema_version: "1.0"`.

## Success

```json
{
  "ok": true,
  "schema_version": "1.0",
  "data": { /* command-specific payload */ }
}
```

## Error

```json
{
  "ok": false,
  "schema_version": "1.0",
  "error": "human-readable message",
  "code": "ENOENT | ENOTFOUND | EVALIDATION | EAMBIGUOUS | EUNKNOWN"
}
```

Exit codes: `0` success · `1` validation/lookup error (most `Exxx`) ·
`2` system error (`ENOENT` for the data file, `EUNKNOWN`).

## Variants on success envelopes

### Dry-run (when `--dry-run` is passed on a mutation)

```json
{
  "ok": true,
  "schema_version": "1.0",
  "data": {
    "dry_run": true,
    "would": { /* same shape as a real success would emit */ }
  }
}
```

### Idempotent replay (same `--idempotency-key` seen within 24h)

```json
{
  "ok": true,
  "schema_version": "1.0",
  "data": {
    "idempotent_replay": true,
    /* original success payload spread in */
  }
}
```

The original `transaction.id` / `debt.id` / etc. is preserved — that's
the whole point: the second call doesn't create a new row.

## Error codes — when each fires

| Code | Meaning | Typical handler |
|---|---|---|
| `ENOENT` | `~/.finance/data.json` missing | Suggest `finance init` |
| `ENOTFOUND` | id not found (transaction, debt, recurring) | List candidates and re-ask |
| `EVALIDATION` | bad input (negative amount, bad date, missing arg) | Explain what was wrong |
| `EAMBIGUOUS` | category prefix matched multiple | Show the matches and ask which |
| `EUNKNOWN` | system error (disk, JSON parse) | Surface verbatim, don't paper over |

## ID conventions

- Transactions: `tx-<uuid>` (legacy `txn-` also accepted).
- Debts: `debt-<slug or uuid>`.
- Categories: `cat-<slug>` (e.g. `cat-food`, `cat-freelance`, `cat-debt`).
- Recurring income/expense: `ri-<slug>` / `re-<slug>`.
- Idempotency keys: caller-supplied strings; reuse the same key for any
  retry of the same logical operation within 24h.

## Schema version handshake

`finance schema --json` returns the canonical command surface AND the
current `schema_version`. If you see a version newer than `"1.0"` in any
envelope, re-call `finance schema --json` and look for shape changes
(new fields are additive and safe; removed fields would only happen on
a major version bump).
