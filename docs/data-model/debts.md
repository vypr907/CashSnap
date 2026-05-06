# 💳 Debts Table

## 🎯 Purpose

The **Debts** table represents all debt accounts tracked in CashSnap.

Examples:

* Credit cards
* Auto loans
* Personal loans
* Installment plans
* Medical debts

Debts act as the **container** for:

* Statements
* Payments
* Allocations
* Charges
* Ledger activity

---

# 🧠 Core Concept

Debts do **not directly track balances**.

Instead:

```text
Ledger → Derived Balances → Debts
```

The Debts table provides:

* context
* classification
* reporting fields

---

# 🧱 Key Columns

## Identity & Classification

| Column    | Description                   |
| --------- | ----------------------------- |
| Row ID    | Primary key                   |
| Debt Name | Display name                  |
| Debt Type | Loan / Credit Card / Personal |
| Model     | Defines behavior (critical)   |
| Pay To    | Who the debt is paid to       |

---

## Financial Inputs

| Column           | Description      |
| ---------------- | ---------------- |
| Starting Balance | Original balance |
| Interest Rate    | Optional         |
| Minimum Payment  | Optional         |

---

## Derived Balances (Important)

These should align with Ledger.

### Principal Balance

```text
Starting Balance - Principal Applied Ledger
```

### Interest Balance

```text
Interest Charges - Interest Payments
```

### Remaining Balance

```text
Principal Balance + Interest Balance
```

---

## Helper / Tracking Columns

| Column                 | Description                                  |
| ---------------------- | -------------------------------------------- |
| Last Statement Balance | For credit cards                             |
| Last Statement End     | Statement tracking                           |
| Paid So Far            | Must be clearly defined (total vs principal) |
| Status                 | Active / Paid / Closed                       |

---

# 🔄 Models (CRITICAL)

The `Model` column drives behavior.

## Loan (Statement-Based)

* Uses Statements
* Uses Loan_Pmt_Splits
* Uses PaymentAllocations
* Interest/Principal split required

---

## Loan (Transaction-Based)

* Driven by transaction feeds
* May not use statements
* Still uses allocations + ledger

---

## Credit Card

* Statement-driven
* Purchases may be modeled as Debt Charges
* Payments reduce statement balance

---

## Installment

* Fixed schedule
* May treat principal as structured obligation
* Can optionally use Debt Charges

---

## Personal Debt

* Simpler model
* May not require allocations
* Often direct ledger-driven

---

# 🔗 Relationships

| Related Table      | Purpose                 |
| ------------------ | ----------------------- |
| Statements         | Defines billing periods |
| Payments           | Cash applied to debt    |
| PaymentAllocations | Splits payments         |
| Debt Charges       | Interest/fees           |
| Ledger             | Source of truth         |

---

# ⚠️ Critical Rules

## 1. Ledger is authoritative

Debts must not independently calculate balances that contradict Ledger.

---

## 2. Principal vs Interest separation

* Principal reduces balance
* Interest does not reduce principal
* Must be tracked separately

---

## 3. Payment amount ≠ Principal reduction

This is one of the most common bugs.

---

## 4. Statement-based loans require strict flow

* Statements define scope
* Allocations define behavior
* Ledger defines impact

---

# ⚠️ Common Issues

| Issue                      | Cause                                   |
| -------------------------- | --------------------------------------- |
| Balance mismatch           | Ledger vs formula inconsistency         |
| Double principal reduction | Full payment + allocation both applied  |
| Interest not clearing      | Missing PaymentAllocation or ledger     |
| Incorrect paid status      | Using wrong metric (total vs principal) |

---

# 🧪 Debug Tips

To debug a Debt:

1. Check Ledger rows for that Debt
2. Filter by:

   * Allocation Type
   * Charge Category
3. Validate:

   * Principal Applied
   * Interest Charges
   * Interest Payments

---

# 🔗 Related Docs

* `docs/data-model/ledger.md`
* `docs/data-model/payment-allocations.md`
* `docs/data-model/debt-charges.md`
* `docs/workflows/payment-allocation-flow.md`
