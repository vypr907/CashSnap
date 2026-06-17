# ADR-0002 — Use Ledger as SSOT for Bill Charge Balances

## Status

Accepted

## Date

2026-06-17

## Context

`Bill Charges[Remaining Amount]` previously calculated the remaining amount from:

- `Bill Charges[Amount]`
- `Related Adjustments[Signed Amount]`
- `Related Transaction Links[Amount Applied]`

That made the formula accurate only as long as all related helper/detail tables remained perfectly aligned. It also meant the balance could bypass Ledger, even though Ledger is intended to be the financial source of truth.

## Decision

`Bill Charges[Remaining Amount]` will calculate from `Ledger[Signed Amount]` using rows linked by `Ledger[Bill Charge ID]`.

Approved formula:

```appsheet
MAX(
  LIST(
    0.0,
    SUM(
      SELECT(
        Ledger[Signed Amount],
        AND(
          [Bill Charge ID] = [_THISROW].[Row ID],
          ISNOTBLANK([Signed Amount])
        )
      )
    )
  )
)
```

## Consequences

### Positive

- One balance source for Bill Charges.
- Returned payments and reversals become simpler because they are just signed ledger events.
- Balance reconciliation becomes easier.
- Reduces drift between transaction allocation detail and displayed remaining amount.

### Negative / Risk

- Historical Bill Charges must be backfilled with original `Charge` ledger rows.
- Ledger row creation must be reliable before the formula is switched in production.
- Incorrect `Signed Amount` values will directly affect balances.

## Required Follow-Up

- Backfill missing original charge ledger rows.
- Add an admin diagnostic slice for Bill Charges where legacy remaining and ledger remaining differ.
- Verify returned payment flows create positive reversal ledger rows tied to the original `Bill Charge ID`.
