# 📊 Ledger Table

## 🎯 Purpose

The **Ledger** is the financial source of truth for CashSnap.

Every meaningful financial event must be represented here.

---

## 🧠 Core Concept

```text
If it affects money, it must exist in Ledger
```

All balances should ultimately derive from:

```text
SUM(Ledger[Signed Amount])
```

---

# 🧱 Table Role in CashSnap

The Ledger:

* Records all financial activity
* Normalizes all flows (Bills, Debts, Payments, Allocations)
* Provides a full audit trail
* Drives all balances (eventually SSOT)

---

# 📋 Column Inventory

<details>
<summary>Click to expand full column list</summary>

| Column           | Type                     | App Formula       | Initial Value | Notes                        |
| ---------------- | ------------------------ | ----------------- | ------------- | ---------------------------- |
| LedgerID         | Text (Key)               |                   | `UNIQUEID()`  | Primary key                  |
| Date             | DateTime                 |                   |               | Event date                   |
| Amount           | Price                    |                   |               | Raw amount                   |
| Signed Amount    | Price (VC)               | [Formula](#-signed-amount-formula) |               | Core financial impact        |
| Type             | Enum                     |                   |               | Defines behavior             |
| Affects Balance? | Yes/No                   |                   |               | Controls inclusion in totals |
| Charge Category  | Enum                     |                   |               | Interest, Fee, etc           |
| Allocation Type  | Enum                     |                   |               | Principal / Interest / Fee   |
| Bill ID          | Ref → Bills              |                   |               | Optional                     |
| Debt ID          | Ref → Debts              |                   |               | Optional                     |
| PaymentID        | Ref → Payments           |                   |               | Source payment               |
| Bill Charge ID   | Ref → Bill Charges       |                   |               | Bill linkage                 |
| Debt Charge ID   | Ref → Debt Charges       |                   |               | Interest/Fee linkage         |
| TransLink        | Ref → Transaction Links  |                   |               | Bill allocation source       |
| PmtAllocID       | Ref → PaymentAllocations |                   |               | Debt allocation source       |
| StatementID      | Ref → Statements         |                   |               | Statement linkage            |
| Src Row ID       | Text                     |                   |               | Originating row              |
| Notes            | Text                     |                   |               | Optional                     |
| Label2           | Text (VC)                | See below         |               | Human-readable label         |

</details>

---

# 🧮 Key Virtual Columns

## 🔽 Signed Amount Formula

<details>
<summary>Click to expand Signed Amount logic</summary>

```appsheet
SWITCH(
  [Type],

  "Charge", [Amount],
  "Debt Charge", [Amount],

  "Bill Payment", -[Amount],
  "Debt Payment", -[Amount],
  "Principal Applied", -[Amount],

  "Bill Payment Reversal", [Amount],
  "Debt Payment Reversal", [Amount],
  "Principal Reversal", [Amount],

  "Adjustment", [Amount],

  0.0
)
```

</details>

---

## 🔽 Label2 Formula (Example Pattern)

<details>
<summary>Click to expand Label2 logic</summary>

```appsheet
CONCATENATE(
  [Type],
  " | ",
  IF(ISNOTBLANK([PaymentID]), [PaymentID].[Payment Name], ""),
  " | ",
  TEXT([Date])
)
```

</details>

---

# 🔄 Ledger Event Types

<details>
<summary>Click to expand event type breakdown</summary>

## Charges (Positive)

| Type        | Description    |
| ----------- | -------------- |
| Charge      | Bill charge    |
| Debt Charge | Interest / Fee |

---

## Payments (Negative)

| Type              | Description          |
| ----------------- | -------------------- |
| Bill Payment      | Bill payment         |
| Debt Payment      | Interest/Fee payment |
| Principal Applied | Principal reduction  |

---

## Reversals (Positive)

| Type                  | Description       |
| --------------------- | ----------------- |
| Bill Payment Reversal | Undo bill payment |
| Debt Payment Reversal | Undo interest/fee |
| Principal Reversal    | Undo principal    |

---

## Adjustments

| Type       | Description       |
| ---------- | ----------------- |
| Adjustment | Manual correction |

</details>

---

# 🤖 Actions

<details>
<summary>Click to expand Ledger-related actions</summary>

Typical actions:

* Add Ledger Row (generic)
* Add Debt Charge Ledger
* Add Payment Ledger
* Add Principal Applied Ledger
* Add Reversal Ledger

Each action should:

* Populate all relevant refs
* Set correct Type
* Set Amount
* Ensure Signed Amount computes correctly

</details>

---

# 🤖 Bots / Processes

<details>
<summary>Click to expand bot interactions</summary>

## LedgerBot

Trigger:

* On Transaction Links add

Creates:

* Bill Payment ledger rows

---

## Pmt Allocation Bot

Trigger:

* On PaymentAllocations add

Creates:

* Principal Applied ledger
* Debt Payment ledger

---

## Debt Charge Bot

Trigger:

* On Debt Charge add

Creates:

* Positive Debt Charge ledger

---

## Reversal Flow

* Reversal Payments create mirrored ledger rows
* Must invert Signed Amount behavior

</details>

---

# 🔗 Relationships

<details>
<summary>Click to expand relationships</summary>

```text
Bills → Bill Charges → Transaction Links → Ledger
Payments → Transaction Links → Ledger
Payments → PaymentAllocations → Ledger
Debts → Debt Charges → Ledger
Statements → PaymentAllocations → Ledger
```

</details>

---

# ⚠️ Critical Rules

<details>
<summary>Click to expand critical rules</summary>

## 1. No duplicate financial impact

Each real-world event should produce exactly one ledger effect.

---

## 2. Allocation overrides full payment

If allocations exist:

* Do NOT apply full payment as balance change

---

## 3. Affects Balance must be correct

Incorrect usage breaks all balances.

---

## 4. Ledger must be complete

Missing rows = broken system

</details>

---

# ⚠️ Known Issues

<details>
<summary>Click to expand known issues</summary>

| Issue                 | Cause                     |
| --------------------- | ------------------------- |
| Balance mismatch      | Missing or duplicate rows |
| Principal incorrect   | Double-counting           |
| Interest not clearing | Missing allocation        |
| Rounding errors       | Precision issues          |

</details>

---

# 🧪 Debug Strategy

<details>
<summary>Click to expand debug workflow</summary>

1. Filter Ledger by Debt ID or Bill ID
2. Group by:

   * Type
   * Allocation Type
3. Validate:

   * Signed Amount totals
   * Expected row counts

Trace using:

* PaymentID
* PmtAllocID
* TransLink
* TraceID (via Debug Log)

</details>

---

# 🔧 Repair Notes

<details>
<summary>Click to expand repair strategies</summary>

* Disable old full-payment ledger rows:

  ```text
  Affects Balance? = FALSE
  ```

* Backfill missing PmtAllocID

* Normalize Charge Category and Allocation Type

* Ensure every PaymentAllocation has exactly one ledger row

</details>

---

# 🔗 Related Docs

* `docs/data-model/debts.md`
* `docs/data-model/payments.md`
* `docs/workflows/payment-allocation-flow.md`
* `docs/workflows/loan-statement-flow.md`
