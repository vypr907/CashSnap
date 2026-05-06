# 🧱 CashSnap Tables Overview

## 🎯 Purpose

This document summarizes the core CashSnap data model, table responsibilities, and major relationships.

CashSnap is designed around a Ledger-first architecture where most financial state should eventually be traceable back to Ledger rows.

---

# 🧠 Core Model

```text
Bills / Debts
→ Charges / Statements
→ Payments
→ Allocations
→ Ledger
→ Balances
```

---

# 📊 Core Tables | [Master Table List](master-tables.md)

| Table              | Primary Role              | Notes                                                  |
| ------------------ | ------------------------- | ------------------------------------------------------ |
| [Bills](bills.md)              | Recurring obligations     | Includes simple due-date and statement-cycle bills     |
| [Bill Charges](bill-charges.md)       | Bill instances            | One charge per bill cycle/period                       |
| [Payments](payments.md)           | Cash movement             | Applies to bills or debts                              |
| [Transaction Links](transaction-links.md)  | Bill payment allocations  | Links Payments to Bill Charges                         |
| [Ledger](ledger.md)             | Financial source of truth | Records all financial effects                          |
| [Debts](debts.md)              | Debt accounts             | Loans, credit cards, personal debts, installments      |
| [Statements](statements.md)         | Statement periods         | Used mainly for credit cards and statement-based loans |
| [PaymentAllocations](payment-allocations.md) | Debt payment splits       | Principal / Interest / Fee allocations                 |
| [Debt Charges](debt-charges.md)       | Debt-side charges         | Interest, fees, penalties, purchases if modeled        |
| Adjustments        | Manual corrections        | Bill/debt corrections and statement credits            |
| Debug Log          | Observability             | Tracks bot/process execution                           |

---
---
---
---
---

# 🔹 [Bills](bills.md)
<details>

## Purpose

Tracks recurring bill obligations.

Examples:

* Internet
* Utilities
* Subscriptions
* Insurance
* Statement-cycle bills

## Key Concepts

Bills may use different billing models:

| Billing Model   | Meaning                                                  |
| --------------- | -------------------------------------------------------- |
| Simple Due-Date | Standard recurring bill based on due date                |
| Statement Cycle | Service period / statement period separate from due date |

## Related Tables

| Related Table     | Relationship                                 |
| ----------------- | -------------------------------------------- |
| Bill Charges      | One Bill → Many Bill Charges                 |
| Payments          | One Bill → Many Payments                     |
| Transaction Links | Payments allocated to Bill Charges           |
| Ledger            | Bill charges and payments create ledger rows |

</details>

---
---

# 🔹 [Bill Charges](bill-charges.md)
<details>

## Purpose

Represents a specific bill obligation for a cycle or period.

Example:

* GCI April 2026
* Rent May 2026
* Insurance Q2 2026

## Key Responsibilities

* Store charge amount
* Track cycle/period
* Connect payments to the correct charge
* Support pay-period dashboard logic

## Related Tables

| Related Table     | Relationship                             |
| ----------------- | ---------------------------------------- |
| Bills             | Each Bill Charge belongs to one Bill     |
| Transaction Links | One Bill Charge → Many Transaction Links |
| Ledger            | One Bill Charge → Many Ledger rows       |
| Adjustments       | Optional corrections                     |

</details>

---
---

# 🔹 [Payments](payments.md)
<details>

## Purpose

Records money paid toward a bill or debt.

Payments are the cash event, not necessarily the final accounting effect.

## Key Responsibilities

* Track payment date, amount, status
* Link to Bill or Debt
* Support reversals/returns
* Trigger allocation flows

## Related Tables

| Related Table      | Relationship                   |
| ------------------ | ------------------------------ |
| Transaction Links  | Bill-side allocation records   |
| PaymentAllocations | Debt-side allocation records   |
| Ledger             | Financial effects              |
| Statements         | Optional statement association |

</details>

---
---

# 🔹 [Transaction Links](transaction-links.md)
<details>

## Purpose

Represents how a bill payment is applied to one or more Bill Charges.

Example:
A $256.50 payment may split across:

* December charge: $131.51
* January charge: $124.99

## Key Responsibilities

* Link Payment → Bill Charge
* Store Amount Applied
* Trigger LedgerBot
* Support split payments

## Related Tables

| Related Table | Relationship                                     |
| ------------- | ------------------------------------------------ |
| Payments      | Each Transaction Link belongs to one Payment     |
| Bill Charges  | Each Transaction Link applies to one Bill Charge |
| Ledger        | Usually creates one ledger row                   |

</details>

---
---

# 🔹 [Ledger](ledger.md)
<details>

## Purpose

Ledger is intended to be the source of truth for financial state.

All meaningful financial events should create Ledger rows.

## Examples of Ledger Events

| Event                 | Ledger Type                           |
| --------------------- | ------------------------------------- |
| Bill charge created   | Charge                                |
| Bill payment applied  | Bill Payment                          |
| Debt interest charged | Debt Charge                           |
| Debt principal paid   | Principal Applied                     |
| Debt interest paid    | Debt Payment                          |
| Reversal              | Payment Reversal / Principal Reversal |
| Adjustment            | Adjustment                            |

## Key Responsibilities

* Store signed financial impact
* Support balances
* Link back to source records
* Provide audit trail

## Important Fields

* Signed Amount
* Type
* Affects Balance?
* Bill ID
* Debt ID
* Bill Charge ID
* Payment ID
* Transaction Link
* PaymentAllocation ID
* Debt Charge ID
* Statement ID
* Charge Category
* Allocation Type

</details>

---
---

# 🔹 [Debts](debts.md)
<details>

## Purpose

Tracks debt accounts and obligations.

Examples:

* Credit cards
* Loans
* Personal debts
* Installment agreements

## Key Concepts

Debts may differ by model:

| Debt Type / Model        | Meaning                                 |
| ------------------------ | --------------------------------------- |
| Credit Card              | Statement-based credit balance          |
| Loan (Statement-Based)   | Loan with statements and split payments |
| Loan (Transaction-Based) | Loan tracked from transaction feeds     |
| Installment              | Fixed repayment schedule                |
| Personal Debt            | Simpler balance tracking                |

## Related Tables

| Related Table      | Relationship                   |
| ------------------ | ------------------------------ |
| Statements         | One Debt → Many Statements     |
| Payments           | One Debt → Many Payments       |
| PaymentAllocations | Debt payment split records     |
| Debt Charges       | Interest/fee obligations       |
| Ledger             | Debt financial source of truth |

</details>

---
---

# 🔹 [Statements](statements.md)
<details>

## Purpose

Represents a statement period for a debt account.

Used for:

* Credit cards
* Statement-based loans

## Key Responsibilities

* Store statement start/end
* Store statement balance
* Attach related payments
* Anchor fees, interest, adjustments, and allocations

## Related Tables

| Related Table      | Relationship                         |
| ------------------ | ------------------------------------ |
| Debts              | Each Statement belongs to one Debt   |
| Payments           | Payments may link to Statements      |
| PaymentAllocations | Allocations may link to Statements   |
| Debt Charges       | Interest/fees may link to Statements |
| Ledger             | Ledger rows may link to Statements   |

</details>

---
---

# 🔹 [PaymentAllocations](payment-allocations.md)
<details>

## Purpose

Represents how a debt payment is applied.

Allocation types:

* Principal
* Interest
* Fee

## Key Responsibilities

* Split payment into accounting categories
* Create/link Debt Charges for interest/fees
* Create ledger rows
* Support reversals

## Related Tables

| Related Table | Relationship                                     |
| ------------- | ------------------------------------------------ |
| Payments      | Each allocation belongs to one Payment           |
| Debts         | Each allocation belongs to one Debt              |
| Statements    | Optional but important for statement-based debts |
| Debt Charges  | Interest/Fee allocations link to charges         |
| Ledger        | Each allocation should create one ledger row     |

</details>

---
---

# 🔹 [Debt Charges](debt-charges.md)
<details>

## Purpose

Represents non-principal debt obligations.

Examples:

* Interest
* Fees
* Penalties
* Credit card purchases, if modeled as charges

## Key Rule

Debt Charges are normally used for:

```text
Interest / Fees / Penalties / Purchases
```

Not for:

```text
Principal
```

Unless principal installments are intentionally modeled as charges.

## Related Tables

| Related Table      | Relationship                                     |
| ------------------ | ------------------------------------------------ |
| Debts              | Each Debt Charge belongs to one Debt             |
| Statements         | Optional statement link                          |
| PaymentAllocations | Interest/Fee allocations may link to charges     |
| Ledger             | Debt Charge creation creates positive ledger row |

</details>

---
---

# 🔹 [Adjustments](adjustments.md)
<details>

## Purpose

Manual corrections or special credits/debits.

Examples:

* Statement credits
* Balance corrections
* Refund adjustments
* Manual reconciliation corrections

## Related Tables

| Related Table | Relationship                                         |
| ------------- | ---------------------------------------------------- |
| Bills         | Optional Bill link                                   |
| Bill Charges  | Optional Bill Charge link                            |
| Debts         | Optional Debt link                                   |
| Statements    | Optional Statement link                              |
| Ledger        | Should create ledger rows where financially relevant |

</details>

---
---

# 🔹 Debug Log

<details>

## Purpose

Tracks automation and process behavior.

Used to answer:

* What ran?
* When did it run?
* Why did it run?
* What row caused it?
* Did it succeed or fail?

## Key Fields

* Timestamp
* Process
* Action
* Stage
* Status
* Source Table
* Message
* Details
* TraceID
* Row Identifier
* Bill ID
* Debt ID
* TransLink ID
* Ledger ID
* ProcessRunID

</details>

---
---

# 🔗 Major Relationships

```text
Bills
└─ Bill Charges
   └─ Transaction Links
      └─ Ledger

Payments
├─ Transaction Links
│  └─ Ledger
└─ PaymentAllocations
   ├─ Debt Charges
   └─ Ledger

Debts
├─ Statements
├─ Debt Charges
├─ PaymentAllocations
└─ Ledger
```

---

# 🧠 Source of Truth Rules

## Ledger should answer:

* What is owed?
* What has been paid?
* What balance remains?
* What happened financially?

## Non-ledger tables should answer:

* What process created the event?
* What object does it belong to?
* What workflow state is it in?
* What user-entered data started the process?

---

# ⚠️ Known Data Model Risks

| Risk                        | Notes                                                          |
| --------------------------- | -------------------------------------------------------------- |
| Duplicate ledger impact     | Same payment can reduce balance twice if not gated             |
| Duplicate debt charges      | Interest/Fee charges can be created by multiple bots           |
| Circular formulas           | Especially in Bills / Bill Charges cycle logic                 |
| Expensive SELECTs           | Can slow AppSheet sync                                         |
| Type mismatch errors        | Price vs Decimal vs Number issues                              |
| Statement timing edge cases | Payment and reversal may belong to different statement periods |

---

# ✅ Documentation Priority

Each major table should eventually have its own document covering:

* Purpose
* Key columns
* Formulas
* Actions
* Bots
* Related workflows
* Known issues
* Repair notes

---

# [Master Table List](master-tables.md)