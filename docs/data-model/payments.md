# 💵 Payments Table

## 🎯 Purpose

The **Payments** table represents cash movement into the system.

Payments:

* Do NOT directly change balances
* Trigger allocation workflows
* Serve as the starting point for financial processing

---

# 🧠 Core Concept

```text
Payment = cash event
Allocation = accounting effect
Ledger = financial truth
```

---

# 🧱 Key Columns

## Identity

| Column       | Description  |
| ------------ | ------------ |
| Row ID       | Primary key  |
| Payment Name | Display name |

---

## Financial

| Column           | Description                |
| ---------------- | -------------------------- |
| Amount Paid      | Payment amount             |
| Date             | Payment date               |
| Effective Amount | May include reversal logic |

---

## Classification

| Column         | Description     |
| -------------- | --------------- |
| Type           | Bill / Debt     |
| Payment Method | Ref to Accounts |

---

## Status

| Column         | Description                  |
| -------------- | ---------------------------- |
| Cleared        | TRUE/FALSE                   |
| Cleared Date   | When finalized               |
| Payment Status | Pending / Cleared / Returned |

---

## Relationships

| Column           | Description        |
| ---------------- | ------------------ |
| Bill ID          | Bill linkage       |
| Debt ID          | Debt linkage       |
| Statement_ID     | Optional statement |
| Reverses Payment | Link to original   |
| Return Date      | For reversals      |

---

## Allocation Helpers

| Column                | Description                |
| --------------------- | -------------------------- |
| Remaining To Allocate | Tracks allocation progress |
| NextChargeID          | For bill allocation        |
| Processed             | Prevents duplicate runs    |
| TriggerSource         | Bot tracking               |

---

# 🔄 Payment Flow

```text
Payment Created
→ Allocation Process Starts
→ Transaction Links OR PaymentAllocations created
→ Ledger entries generated
→ Payment fully allocated
```

---

# 🔁 Reversal Flow

```text
Reverse Payment Created
→ Mirror allocations created
→ Ledger reversal rows created
→ Original payment marked returned
```

---

# ⚠️ Critical Rules

## 1. Payments do not directly affect balances

They must go through:

* Transaction Links (Bills)
* PaymentAllocations (Debts)

---

## 2. Cleared status matters

Only cleared payments should affect financial state.

---

## 3. Allocation must fully consume payment

```text
Remaining To Allocate = 0
```

---

## 4. Reversals must mirror original allocations

---

# ⚠️ Common Issues

| Issue                    | Cause                    |
| ------------------------ | ------------------------ |
| Payment not applied      | Allocation not triggered |
| Partial allocation stuck | Remaining not processed  |
| Double allocation        | Bot re-trigger           |
| Wrong statement linkage  | Date mismatch            |

---

# 🧪 Debug Tips

For a Payment:

1. Check:

   * Transaction Links (Bills)
   * PaymentAllocations (Debts)

2. Verify:

   * Remaining To Allocate = 0
   * Ledger rows exist

3. Trace using:

   * TraceID
   * Debug Log

---

# 🔗 Related Docs

* `docs/data-model/transaction-links.md`
* `docs/data-model/payment-allocations.md`
* `docs/workflows/new-payment-flow.md`
