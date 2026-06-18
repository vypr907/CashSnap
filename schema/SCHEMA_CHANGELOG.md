# 📚 SCHEMA_CHANGELOG.md

> Last Updated: 2026-06-18
>
> Purpose: Append-friendly record of CashSnap schema, formula, automation, and documentation changes.
>
> Related Docs:
>
> - [AI_CONTEXT.md](./AI_CONTEXT.md)
> - [cashsnap-schema.yml](./schema/cashsnap-schema.yml)
> - [ledger.md](./docs/tables/ledger.md)
> - [bill-charges.md](./docs/tables/bill-charges.md)
> - [payment-processor-bot.md](./docs/bots/payment-processor-bot.md)
> - [payment-completion-bot.md](./docs/bots/payment-completion-bot.md)
> - [pmt-allocation-bot.md](./docs/bots/pmt-allocation-bot.md)

---

# 📌 Overview

This changelog tracks meaningful changes to the CashSnap AppSheet schema and system behavior.

Use this file for:

- Table additions/removals
- Column additions/removals/renames
- Formula changes
- Initial Value / Valid If changes
- Bot/action/process changes
- Ledger SSOT contract changes
- Migration/backfill requirements
- Documentation standard changes

This is not a user-facing release log. It is a project-maintenance log for AI tools, GitHub, and future debugging.

---

# 🧠 Changelog Rules

<details open>
<summary><strong>Expand Changelog Rules</strong></summary>

Each entry should include:

- **Date**
- **Summary**
- **Changed**
- **Fixed**
- **Migration / Repair Notes**
- **Related Docs**

Preferred format:

```md
## YYYY-MM-DD — Short Title

### Summary

### Changed

### Fixed

### Migration / Repair Notes

### Related Docs
```

Rules:

- Append new entries at the top under `# 🗓️ Change Log`.
- Do not delete historical entries unless they were committed in error.
- Mention exact table/column/bot/action names whenever possible.
- Include AppSheet formulas when formula behavior changes.
- Add a migration note when historical rows need repair or backfill.

</details>

---

# 🗓️ Change Log

## 2026-06-18 — Create schema baseline and changelog

### Summary

Created the first dedicated CashSnap schema snapshot and schema changelog files.

### Changed

- Added `schema/cashsnap-schema.yml` as the canonical machine-readable schema/context snapshot for AI project sources.
- Added `SCHEMA_CHANGELOG.md` as the append-friendly schema and workflow change history.
- Captured current known CashSnap tables, core columns, workflow rules, bots, Ledger sign conventions, known issues, and current priorities.

### Fixed

- Closed the documentation gap where the project had an `AI_CONTEXT.md` but no structured schema file or schema changelog.

### Migration / Repair Notes

- Treat `cashsnap-schema.yml` as a documented baseline, not a raw AppSheet export.
- Next recommended improvement is to compare this file against the live AppSheet table/column list and fill any missing columns.

### Related Docs

- `AI_CONTEXT.md`
- `schema/cashsnap-schema.yml`

---

## 2026-06-18 — Standardize Bill Charges on Ledger SSOT

### Summary

Updated the Bill Charges balance model so `Bill Charges[Remaining Amount]` derives from `Ledger[Signed Amount]` rather than directly summing `Adjustments` and `Transaction Links`.

### Changed

- `Bill Charges[Remaining Amount]` should use Ledger SSOT.
- `Bill Charges[Amount]` becomes descriptive/input data for the original charge once Ledger SSOT is active.
- `Transaction Links[Amount Applied]` remains allocation detail.
- `Adjustments[Signed Amount]` remains adjustment detail.
- Returned payments, credits, reversals, and corrections must affect Bill Charge balance only through Ledger rows tied to the same `Bill Charge ID`.

### Formula

```appsheet
IF(
  SUM(
    SELECT(
      Ledger[Signed Amount],
      AND(
        [Bill Charge ID] = [_THISROW].[Row ID],
        ISNOTBLANK([Signed Amount])
      )
    )
  ) > 0,
  SUM(
    SELECT(
      Ledger[Signed Amount],
      AND(
        [Bill Charge ID] = [_THISROW].[Row ID],
        ISNOTBLANK([Signed Amount])
      )
    )
  ),
  0
)
```

### Fixed

- Replaced the earlier `MAX(LIST(0.0, SUM(...)))` pattern because AppSheet can treat `0.0` as Decimal and `SUM(Ledger[Signed Amount])` as Price, causing:

```text
LIST has elements of mismatched types
```

### Migration / Repair Notes

- Before production cutover, backfill one positive `Ledger` row of `Type = "Charge"` for every existing `Bill Charges` row that does not already have one.
- Validate old remaining amount vs ledger-derived remaining amount before enabling the final formula.
- Do not filter this formula by `IsCashMovement`; original charges and adjustments affect balances even when they are not cash movement.

### Related Docs

- `docs/tables/bill-charges.md`
- `docs/tables/ledger.md`
- `docs/decisions/ADR-0002-ledger-ssot-for-bill-charge-balances.md`

---

## 2026-06-18 — Rename New Payment Bot to Payment Processor Bot

### Summary

Documented the previously named `New Payment Bot` as `Payment Processor Bot`.

### Changed

- `Payment Processor Bot` is now the preferred name for the bot that begins and continues the payment processing loop.
- The bot works together with `Payment Completion Bot`.
- The processor should create allocation detail rows, not directly mutate balances.

### Fixed

- Clarified that bill payments flow through `Transaction Links` and debt payments flow through `PaymentAllocations` before Ledger impact.

### Migration / Repair Notes

- Rename bot docs and references from `New Payment Bot` to `Payment Processor Bot` where possible.
- Keep a note in docs that the previous name was `New Payment Bot` for historical traceability.

### Related Docs

- `docs/bots/payment-processor-bot.md`
- `docs/bots/payment-completion-bot.md`
- `docs/tables/payments.md`

---

## 2026-06-18 — Add Payment Completion Bot documentation

### Summary

Added documentation for the companion bot that checks whether payment processing is complete or needs to loop again.

### Changed

- Added `Payment Completion Bot` as the finalization/re-loop companion to `Payment Processor Bot`.
- Documented responsibilities:
  - validate remaining payment amount
  - verify allocation/ledger rows
  - mark `Payments[Processed]` when complete
  - re-trigger/continue processing if unallocated amount remains
  - log failure states when no target remains

### Fixed

- Clarified separation between allocation creation and completion/finalization.

### Migration / Repair Notes

- Ensure `Payments[Processed]` is only set when allocation and downstream ledger expectations are satisfied.
- Ensure Debug Log rows share a stable `TraceID`, usually the `PaymentID`.

### Related Docs

- `docs/bots/payment-completion-bot.md`
- `docs/bots/payment-processor-bot.md`
- `docs/tables/debug-log.md`

---

## 2026-06-18 — Refresh AI context and project source guidance

### Summary

Created an updated standalone `AI_CONTEXT.updated.md` and clarified which docs should be loaded into project sources.

### Changed

- Consolidated current CashSnap architecture, workflows, schema notes, Ledger SSOT decisions, bot changes, documentation standards, and known issues into a single AI context file.
- Recommended loading `AI_CONTEXT.updated.md`, schema snapshot, core table docs, bot docs, process docs, architecture/ADR docs, and templates into project sources.

### Fixed

- Reduced risk of stale duplicated context by recommending one current AI context file and one current schema file.

### Migration / Repair Notes

- Remove outdated `AI_CONTEXT` drafts from Project Sources once the updated file is loaded.
- Keep `AI_CONTEXT.md`, `cashsnap-schema.yml`, and `SCHEMA_CHANGELOG.md` updated together.

### Related Docs

- `AI_CONTEXT.md`
- `schema/cashsnap-schema.yml`
- `SCHEMA_CHANGELOG.md`

---

## 2026-06-18 — Replace remembered table documentation template

### Summary

Updated the default CashSnap table documentation template to the existing emoji-header `<table>.md` template.

### Changed

The default table doc structure now uses:

- `# 📋 {TABLE_NAME}.md`
- `# 📌 Overview`
- `# 🧠 Design Philosophy`
- `# 🗂️ Table Relationships`
- `# 🔑 Core Columns`
- `# 📚 ALL Columns`
- `# ⚙️ Virtual Columns`
- `# 🧮 Formula Reference`
- `# 🔄 Automation Dependencies`
- `# 🧪 Testing Checklist`
- `# 🐞 Known Issues / Edge Cases`
- `# 🚀 Future Improvements`
- `# 🔍 Cross References`
- `# 📚 Change History`

### Fixed

- Replaced the previously extracted lowercase/kebab-case table template preference with the actual project-standard template.

### Migration / Repair Notes

- Future table docs should use the emoji-header template unless the user explicitly asks otherwise.

### Related Docs

- `docs/templates/table.md`
- `docs/tables/bill-charges.md`

---

## 2026-05-08 — Loan statement repair baseline

### Summary

Established the repair model for `Loan (Statement-Based)` debts.

### Changed

- `Loan_Pmt_Splits` are split instruction rows.
- `PaymentAllocations` are applied split records.
- `Debt Charges` represent interest/fee owed.
- `Ledger` is the financial source of truth.
- Principal allocations create `Principal Applied` ledger rows only for statement-based loans.
- Interest/Fee allocations create or link `Debt Charges` and create payment-side ledger rows.

### Fixed

- Disabled the old `Split Happens` path because it created duplicate Interest `Debt Charges` from `Loan_Pmt_Splits`.
- Added or planned missing `Principal Applied` ledger generation.

### Migration / Repair Notes

- Continue repairing missing interest payment ledger rows.
- Populate missing `Allocation Type`, `Charge Category`, `Debt Charge ID`, `PmtAllocID`, and `StatementID` on affected Ledger rows.
- Neutralize legacy full-payment `Debt Payment` ledger rows if split ledgers exist.

### Related Docs

- `docs/processes/loan-statement-process-repair-plan.md`
- `docs/bots/pmt-allocation-bot.md`
- `docs/tables/paymentallocations.md`
- `docs/tables/ledger.md`

---

# 🧪 Schema Validation Checklist

<details open>
<summary><strong>Expand Validation Checklist</strong></summary>

Use this after schema changes:

- [ ] `AI_CONTEXT.md` updated
- [ ] `schema/cashsnap-schema.yml` updated
- [ ] `SCHEMA_CHANGELOG.md` updated
- [ ] Affected table `.md` doc updated
- [ ] Affected bot/process doc updated
- [ ] Any formula changes tested in AppSheet editor
- [ ] No `MAX(LIST(0.0, PriceValue))` Price/List mismatch patterns introduced
- [ ] Ledger sign convention still valid
- [ ] No duplicate financial impact introduced
- [ ] Debug Log traceability preserved
- [ ] Migration/backfill requirements documented

</details>

---

# 🔍 Cross References

## Related Tables

- Ledger
- Payments
- Transaction Links
- PaymentAllocations
- Bills
- Bill Charges
- Debts
- Debt Charges
- Statements
- Loan_Pmt_Splits
- Adjustments
- Debug Log
- Context
- Installment Schedule

---

## Related Bots

- Payment Processor Bot
- Payment Completion Bot
- LedgerBot
- Pmt Allocation Bot
- Loan Statement Bot
- Statement Bot
- Payment Repair Bot
- New Debt Charge Bot
- George Bot

---

## Related Systems

- Ledger SSOT
- Bill Payment Flow
- Debt Payment Flow
- Loan Statement Flow
- Returned Payment / Reversal Flow
- Debug Log Observability
- Documentation Standards
