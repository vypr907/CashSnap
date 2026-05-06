# 🧾 Statements Table

## 🎯 Purpose

The **Statements** table defines billing periods for debts.

Used primarily for:

* Credit cards
* Statement-based loans

---

# 🧠 Core Concept

Statements define:

```text
Time window → Payment scope → Allocation context
```

---

# 🧱 Key Columns

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

# 🔗 Related Docs

* `docs/data-model/debts.md`
* `docs/workflows/loan-statement-flow.md`
* `docs/workflows/payment-allocation-flow.md`
* `docs/workflows/reversals-and-returns.md`
