# statement-bot.md

> Last Updated: 2026-05-07  
> Related Docs:
>
> - [loan-statement-process-repair-plan.md](./loan-statement-process-repair-plan.md)
> - [payment-allocation-process.md](./payment-allocation-process.md)
> - [ledger-system.md](../architecture/ledger-system.md)

---

# Overview

The `Statement Bot` processes newly-added `Statements` rows for supported debt models.

Its primary responsibilities are:

- Attaching orphaned payments to the correct statement
- Creating statement-related charges
- Running reconciliation logic
- Logging statement processing activity
- Delegating specialized loan processing to the `Loan Statement Bot`

---

# Important Design Rule

## Loan (Statement-Based) Debts

The general `Statement Bot` **must NOT process**:

```text
Loan (Statement-Based)
```

Those debts are exclusively handled by:

```text
Loan Statement Bot
```

This prevents:

- Duplicate Interest Debt Charges
- Duplicate ledger creation
- Duplicate payment attachment
- Allocation conflicts
- Double-counted balances

---

# Bot Condition

## Current Recommended Condition

```appsheet
OR(
  ISBLANK([Debt_ID].[Model]),
  [Debt_ID].[Model] <> "Loan (Statement-Based)"
)
```

---

# Supported Models

## General Statement Bot SHOULD Handle

- Credit Card
- Credit Card (Statement-Based)
- Legacy statement debts
- Other non-loan statement models

---

## General Statement Bot SHOULD NOT Handle

- Loan (Statement-Based)

These are owned by:

```text
Loan Statement Bot
```

See:
- [Loan Statement Process Repair Plan](./loan-statement-process-repair-plan.md#phase-2--clean-up-statement-bot-ownership)
- [Loan Statement Bot](loan-statement-bot.md)

---

# High-Level Flow

<details>
<summary><strong>Statement Bot Flow</strong></summary>

```text
Statement Added
    ↓
Statement Bot Triggered
    ↓
Debug Log: BOT START
    ↓
Find Payments Missing Statement_ID
    ↓
Attach Payments to Statement
    ↓
Debug Log: Payment Attachment
    ↓
Run Reconciliation / Statement Logic
    ↓
Debug Log: BOT FINISH
```

</details>

---

# Payment Attachment Logic

## Purpose

Attach orphaned payments to the correct statement period.

---

## Typical Criteria

```appsheet
AND(
  [Debt ID] = [_THISROW].[Debt_ID],
  [Date] >= [_THISROW].[Statement_Start],
  [Date] <= [_THISROW].[Statement_End],
  ISBLANK([Statement_ID])
)
```

---

# Debug Logging

The bot should log:

- BOT START
- Payments attached
- Reconciliation start/finish
- Failures
- BOT FINISH

Using the standard Debug Log format:

```text
[Process] | [Stage] | [Status] | [SourceTable]:[Row Identifier] | [Summary]
```

---

# Relationship to Loan Statement Bot

## Ownership Boundary

| Responsibility | Statement Bot | Loan Statement Bot |
|---|---|---|
| Attach CC payments | YES | NO |
| Attach loan payments | NO | YES |
| Create PaymentAllocations | NO | YES |
| Create loan Interest Charges | NO | YES |
| Create Principal Applied ledgers | NO | YES |
| Lock processing Context | NO | YES |

---

# Known Historical Issues

<details>
<summary><strong>Duplicate Interest Charge Issue</strong></summary>

Historically, both:

```text
Loan_Pmt_Splits
```

and

```text
PaymentAllocations
```

created Interest Debt Charges.

This caused:

- Duplicate Interest Charges
- Duplicate Ledgers
- Incorrect Remaining Balance calculations
- Inflated Interest_Balance

The repair plan disables the old:

```text
Split Happens
```

Interest charge path.

</details>

---

# Current Repair Status

## Completed

- Split Happens bot disabled
- Principal Applied ledgers created for existing allocations
- Statement Bot isolated from Loan (Statement-Based)

---

# Future Improvements

- Unified reconciliation framework
- Shared statement-processing helpers
- Automated orphan repair mode
- Context-aware retry handling
- Ledger integrity validation

---

# Related Processes

- Loan Statement Bot
- Payment Repair Bot
- Pmt Allocation Bot
- New Debt Charge Bot
- LedgerBot

---

# References

- Loan Statement Process Repair Plan
- Ledger Repair Actions
- Payment Allocation System
- Statement Reconciliation Logic