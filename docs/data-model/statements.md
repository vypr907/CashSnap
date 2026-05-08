# 🧾 Statements Table

> Last Updated: 2026-05-08
>
> Related Docs:
>
> - [loan-statement-process-repair-plan.md](../processes/loan-statement-process-repair-plan.md)
> - [statement-bot.md](../bots/statement-bot.md)
> - [pmt-allocation-bot.md](../bots/pmt-allocation-bot.md)

---
# 📌 Overview

## 🎯 Purpose

The **Statements** table defines billing periods for debts.

Used primarily for:

* Credit cards
* Statement-based loans

---

# 🧠 Design Philosophy

<details open>
<summary><strong>Expand Design Notes</strong></summary>

## 🧠 Core Concept

Statements define:

```text
Time window → Payment scope → Allocation context
```

## Credit Card Statements

Credit card statements reconcile against:

```text
Statement_Balance
```

using:

```text
Previous Balance
+ Purchases
+ Fees
+ Interest
- Payments
± Adjustments
```

---

## Loan (Statement-Based)

Loan statements reconcile using:

```text
Previous Principal Balance
- Principal Paid
```

while:

- Interest is tracked through Loan_Pmt_Splits
- PaymentAllocations create applied split records
- Debt Charges store interest owed
- Ledger stores financial movement

---

## Ownership Boundary

| System | Responsibility |
|---|---|
| Statements | Statement periods |
| Loan_Pmt_Splits | Split instructions |
| PaymentAllocations | Applied split records |
| Debt Charges | Charges owed |
| Ledger | Financial source of truth |

</details>

---

# 🔑 Key Columns

<details open>
<summary><strong>Expand Key Columns</strong></summary>

## Identity

| Column  | Description |
| ------- | ----------- |
| Row ID  | Primary key |
| Debt_ID | Linked debt |

---

## Period

| Column          | Description |
| --------------- | ----------- |
| Statement_Start | Start date  |
| Statement_End   | End date    |

---

## Financial Inputs

| Column           | Description        |
| ---------------- | ------------------ |
| Previous Balance | Starting balance   |
| New Charges      | Purchases          |
| Fees             | Fee total          |
| Interest         | Interest total     |
| Payments         | Payments in period |
| New Balance      | Ending balance     |

</details>

---

# 📚 ALL Columns

<details open>
<summary><strong>Expand Full Column Reference</strong></summary>

<details open>
<summary><strong>Table</strong></summary>

## 📋 Statements Table — Column Reference

| Column                     | Type           | App Formula               | Initial Value                     | Notes                                                      |
| -------------------------- | -------------- | ------------------------- | --------------------------------- | ---------------------------------------------------------- |
| Row ID                     | Text (Key)     |                           | `UNIQUEID("PackedUUID")`          | Primary key                                                |
| Title                      | Text           |                           |                                   | Legacy/custom title field                                  |
| Statement_Start            | Date           |                           | `TODAY()`                         | Statement start date                                       |
| Statement_End              | Date           |                           | `TODAY()`                         | Statement end date                                         |
| Debt_ID                    | Ref → Debts    |                           | `LOOKUP([Account_ID],"Accounts","Row ID","Linked Debt")` | Linked debt                         |
| Account_ID                 | Ref → Accounts |                           |                                   | Linked account                                             |
| Statement_Balance          | Price          |                           |                                   | Ending statement balance                                   |
| Prev_Bal                   | Price          |                           |                                   | Previous statement balance                                 |
| New_Charges                | Price          |                           |                                   | Manual charge total input                                  |
| Fees                       | Price          |                           |                                   | Statement fees                                             |
| Interest                   | Price          |                           |                                   | Statement interest                                         |
| Min_Pmt                    | Price          |                           |                                   | Minimum payment due                                        |
| Fee Description            | Text           |                           |                                   | Optional fee notes                                         |
| Statement_Type             | Enum           | `LOOKUP([Debt_ID],"Debts","Row ID","Debt_Type")` |            | Derived debt type                                          |
| Related Adjustments        | List           |                           |                                   | Reverse ref/helper list                                    |
| Related Debt Charges       | List           |                           |                                   | Reverse ref/helper list                                    |
| Related Payments           | List           |                           |                                   | Reverse ref/helper list                                    |
| Related PaymentAllocations | List           |                           |                                   | Reverse ref/helper list                                    |
| Related Contexts           | List           |                           |                                   | Linked processing contexts                                 |
| Payments in Cycle          | List           | `SELECT(Payments[Row ID],AND([Debt ID] = [_THISROW].[Debt_ID],[Date] >= [_THISROW].[Statement_Start],[Date] <= [_THISROW].[Statement_End]))` | | Payments within statement window |
| Related Loan_Pmt_Splits    | List           |                           |                                   | Linked loan split rows                                     |
| Principal (V)              | Price          | `SUM(SELECT(Loan_Pmt_Splits[principal_amt],[StatementID] = [_THISROW].[Row ID]))` | | Total principal from splits          |
| Interest (V)               | Price          | `SUM(SELECT(Loan_Pmt_Splits[interest_amt],[StatementID] = [_THISROW].[Row ID]))`  | | Total interest from splits           |
| Title 2.0                  | Text (Label)   | `CONCATENATE(LOOKUP([Account_ID],"Accounts","Row ID","Name")," - ",TEXT([Statement_End],"MMM YYYY"))` | | Current display label |
| Payments Total             | Price          | `SUM(SELECT(Payments[Effective Amount],[Statement_ID] = [_THISROW].[Row ID]))` | | Payments tied to statement              |
| Adjustments Total          | Price          | `SUM(SELECT(Adjustments[Signed Amount],[Statement_ID] = [_THISROW].[Row ID]))` | | Adjustment totals                       |
| Derived New Charges        | Price          | `[Statement_Balance]- [Prev_Bal]- [Interest Total]- [Fees Total]- [Adjustments Total]+ [Payments Total]` | | Derived purchase/charge amount |
| Related Ledgers            | List           |                           |                                   | Reverse ref/helper list                                    |
| Interest Charge Count      | Number         | `COUNT(SELECT(Debt Charges[Row ID],AND([Debt ID] = [_THISROW].[Debt_ID],[Statement_ID] = [_THISROW].[Row ID],[ChargeType] = "Interest")))` | | Count of interest charges |
| Fee Charge Count           | Number         | `COUNT(SELECT(Debt Charges[Row ID],AND([Debt ID] = [_THISROW].[Debt_ID],[Statement_ID] = [_THISROW].[Row ID],[ChargeType] = "Fee")))` | | Count of fee charges |
| Missing Interest Charge?   | Yes/No         | `AND([Interest] > 0,[Interest Charge Count] = 0)` |           | Integrity helper                                           |
| Missing Fee Charge?        | Yes/No         | `AND([Fees] > 0,[Fee Charge Count] = 0)` |                    | Integrity helper                                           |
| Has Duplicate Generated    | Yes/No         | `OR([Interest Charge Count] > 1,[Fee Charge Count] > 1,[Statement Charge Count] > 1)` | | Duplicate detection              |
| Needs Processing?          | Yes/No         | `OR([Missing Interest Charge?],[Missing Fee Charge?],[Missing Statement Charge?])` | | Processing/integrity flag           |
| Debug Status               | Text           | `IFS([Has Duplicate Generated], "Duplicate Charges",[Needs Processing?], "Missing Charges",TRUE, "Complete")` | | Human-readable processing state |
| Interest Total             | Price          | `SUM(SELECT(Debt Charges[Amount],AND([Statement_ID] = [_THISROW].[Row ID],[ChargeType] = "Interest")))` | | Total generated interest charges |
| Fees Total                 | Price          | `SUM(SELECT(Debt Charges[Amount],AND([Statement_ID] = [_THISROW].[Row ID],[ChargeType] = "Fee")))` | | Total generated fee charges |
| Statement Charges Total    | Price          | `SUM(SELECT(Debt Charges[Amount],AND([Statement_ID] = [_THISROW].[Row ID],[ChargeType] = "Statement")))` | | Total generated statement charges |
| Expected Statement Balance | Price          | `[Prev_Bal]+ [Statement Charges Total]+ [Interest Total]+ [Fees Total]+ [Adjustments Total]- [Payments Total]` | | Calculated expected balance |
| Balance Delta              | Number         | `[Statement_Balance]- [Expected Statement Balance]` |         | Reconciliation variance                                    |
| Statement Status           | Text           | `IFS(ISBLANK([Statement_Balance]), "Missing Statement Balance",[Has Duplicate Generated], "Duplicate Charges",[Needs Processing?], "Missing Charges",ABS([Balance Delta]) <= 0.01, "Balanced",ABS([Balance Delta]) <= 5.00, "Minor Variance",TRUE,"Mismatch")` | | Reconciliation status |
| Statement Charge Count     | Number         | `COUNT(SELECT(Debt Charges[Row ID],AND([Debt ID] = [_THISROW].[Debt_ID],[Statement_ID] = [_THISROW].[Row ID],[ChargeType] = "Statement")))` | | Count of statement charges |
| Missing Statement Charge   | Yes/No         | `AND([Derived New Charges] > 0.01,[Statement Charge Count] = 0)` | | Missing statement charge detector                     |
| Statement Charges Delta    | Number         | `[Derived New Charges] - [Statement Charges Total]` |         | Difference between derived and generated statement charges |
| Statement Reconciled?      | Yes/No         | `AND([Statement Status] = "Balanced",NOT([Needs Processing?]),NOT([Has Duplicate Generated]),ABS([Balance Delta]) <= 0.01)` | | Final reconciliation flag |


</details>

## Statement_Balance

### Purpose

Stores the ending balance from the statement.

### Notes

- Used for reconciliation
- Required for Statement Status calculations

---

## Derived New Charges

### Purpose

Calculates inferred purchases/charges for credit-card-style statements.

### Formula

```appsheet
SWITCH(
  [Debt_ID].[Model],

  "Loan (Statement-Based)",
    0,

  [Statement_Balance]
  - [Prev_Bal]
  - [Interest Total]
  - [Fees Total]
  - [Adjustments Total]
  + [Payments Total]
)
```

---

## Expected Statement Balance

### Purpose

Calculates the expected statement balance.

### Formula

```appsheet
SWITCH(
  [Debt_ID].[Model],

  "Loan (Statement-Based)",
    [Prev_Bal] - [Principal (V)],

  [Prev_Bal]
  + [Statement Charges Total]
  + [Interest Total]
  + [Fees Total]
  + [Adjustments Total]
  - [Payments Total]
)
```

---

## Statement Status

### Purpose

Human-readable reconciliation state.

### States

- Balanced
- Minor Variance
- Missing Charges
- Duplicate Charges
- Mismatch

---

## Statement Reconciled?

### Purpose

Final integrity check.

### Formula

```appsheet
AND(
  [Statement Status] = "Balanced",
  NOT([Needs Processing?]),
  NOT([Has Duplicate Generated]),
  ABS([Balance Delta]) <= 0.01
)
```

---

## ⚙️ Virtual Columns

<details open>
<summary><strong>Expand Virtual Columns</strong></summary>

- Payments in Cycle
- Principal (V)
- Interest (V)
- Payments Total
- Adjustments Total
- Derived New Charges
- Interest Total
- Fees Total
- Expected Statement Balance
- Balance Delta
- Statement Status

</details>

</details>

---

## Relationships

| Column             | Description                     |
| ------------------ | ------------------------------- |
| Payments           | Payments linked to statement    |
| PaymentAllocations | Allocations tied to statement   |
| Debt Charges       | Interest/fees tied to statement |
| Ledger             | Financial events                |

---

# 🔄 Statement Flow

```text
Statement Created
→ Payments attached
→ Allocations created
→ Debt Charges created
→ Ledger entries generated
```

---

# 🔹 Payment Attachment Logic

Payments within range:

```appsheet
SELECT(
  Payments[Row ID],
  AND(
    [Debt ID] = [_THISROW].[Debt_ID],
    [Date] >= [_THISROW].[Statement_Start],
    [Date] <= [_THISROW].[Statement_End]
  )
)
```

---

# 🔹 Allocation Context

Statements provide:

* grouping for allocations
* grouping for charges
* reference for ledger entries

---

# ⚠️ Critical Rules

## 1. Statements define allocation scope

Allocations must align with statement period.

---

## 2. Timing mismatches are valid

Example:

* Payment in December
* Reversal in January

System must support cross-statement behavior.

---

## 3. Statement totals must reconcile

```text
Previous Balance
- Payments
+ Purchases
+ Fees
+ Interest
= New Balance
```

---

# ⚠️ Common Issues

| Issue               | Cause                   |
| ------------------- | ----------------------- |
| Payments not linked | Bot not attaching       |
| Incorrect totals    | Missing adjustments     |
| Allocation mismatch | Wrong statement linkage |
| Reversal mismatch   | Cross-period timing     |

---

# 🔄 Automation Dependencies

<details open>
<summary><strong>Expand Automation Dependencies</strong></summary>

## Bots

- Statement Bot
- Loan Statement Bot
- Payment Repair Bot
- Pmt Allocation Bot

---

## Actions

- Set Statement on Payment
- Allocate Payments for Cycle
- Lock Context
- Repair Missing Fields

---

## Processes

- Statement Reconciliation
- Loan Statement Processing
- Payment Allocation

</details>

---

# 🧪 Testing Checklist

<details open>
<summary><strong>Expand Testing Checklist</strong></summary>

## Credit Card Tests

- [ ] Derived New Charges reconciles correctly
- [ ] Statement Charges reconcile correctly
- [ ] Payments Total reconciles correctly

---

## Loan Statement Tests

- [ ] Principal (V) reconciles correctly
- [ ] Interest (V) reconciles correctly
- [ ] Expected Statement Balance reconciles correctly
- [ ] No duplicate interest charges created

---

## Integrity Tests

- [ ] Balance Delta near zero
- [ ] Statement Status correct
- [ ] No orphan PaymentAllocations
- [ ] No orphan Debt Charges
- [ ] No orphan Ledgers

</details>

---

# 🐞 Known Issues / Edge Cases

<details open>
<summary><strong>Expand Known Issues</strong></summary>

- Duplicate Interest Charges
- Missing Statement Charges
- Reversal timing mismatch
- Statement overlap issues
- Payment attachment race conditions

</details>

---

# 🚀 Future Improvements

<details>
<summary><strong>Expand Future Improvements</strong></summary>

- [ ] Add statement integrity validator
- [ ] Improve loan reconciliation workflow
- [ ] Reduce expensive SELECT formulas
- [ ] Improve statement repair tooling

</details>

---

# 🔍 Cross References

## Related Tables

- [Payments](payments.md)
- [PaymentAllocations](payment-allocations.md)
- [Debt Charges](debt-charges.md)
- [Ledger](ledger.md)
- Loan_Pmt_Splits

---

## Related Systems

- Statement Reconciliation
- Loan Statement Process Repair Plan
- Payment Allocation System

---

# 🧪 Debug Tips

For a Statement:

1. Check:

   * Attached payments
   * PaymentAllocations
   * Debt Charges

2. Validate:

   * Ledger rows exist
   * Totals match expected values

---

# 📚 Change History

<details open>
<summary><strong>Expand Change History</strong></summary>

| Date | Change |
|---|---|
| 2026-05-08 | Refactored Statements documentation using standardized table template |
| 2026-05-08 | Added support notes for Loan (Statement-Based) reconciliation |
| 2026-05-08 | Added dual-model reconciliation guidance |

</details>

---

# 🔗 Related Docs

* `docs/data-model/debts.md`
* `docs/workflows/loan-statement-flow.md`
* `docs/workflows/payment-allocation-flow.md`
* `docs/workflows/reversals-and-returns.md`
