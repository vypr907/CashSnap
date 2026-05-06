# 📊 Ledger Table

---

## purpose

The **Ledger** is the financial source of truth for CashSnap.

Every meaningful financial event must be represented here.

---

## core-concept

```text
If it affects money, it must exist in Ledger
```

All balances should ultimately derive from:

```text
SUM(Ledger[Signed Amount])
```

---

## table-role

The Ledger:

* Records all financial activity
* Normalizes all flows (Bills, Debts, Payments, Allocations)
* Provides a full audit trail
* Drives all balances (target: SSOT)

---

## column-inventory

<details>
<summary>View Column Inventory</summary>

| Column             | Type                                  | App Formula                       | Initial Value | Notes                        |
| ------------------ | ------------------------------------- | --------------------------------- | ------------- | ---------------------------- |
| Row ID             | Text                                  |                                   | `UNIQUEID()`  | Generated key                |
| Label              | Text                                  | [Formula](#label-formula)         | `UNIQUEID()`  | App label                    |
| LedgerID           | Text (Key)                            |                                   | `[Row ID]`    | Primary key                  |
| Date               | Date                                  |                                   | `TODAY()`     | Event date                   |
| Amount             | Price                                 |                                   |               | Raw amount                   |
| Signed Amount      | Price (VC)                            | [Formula](#signed-amount-formula) |               | Core financial impact        |
| Type               | Enum                                  |                                   |               | Drives behavior              |
| Affects Balance?   | Yes/No (VC)                           |                                   |               | Controls inclusion in totals |
| Charge Category    | Enum                                  |                                   |               | Interest, Fee, etc           |
| Allocation Type    | Enum                                  |                                   |               | Principal / Interest / Fee   |
| Bill ID            | Ref → [Bills](bills.md)               |                                   |               | Optional                     |
| Debt ID            | Ref → [Debts](debts.md)               |                                   |               | Optional                     |
| PaymentID          | Ref → [Payments](payments.md)         |                                   |               | Source payment               |
| Adjustment ID      | Ref → [Payments](adjustments.md)      |                                   |               | Source Adjustment            |
| Bill Charge ID     | Ref → [Bill Charges](bill-charges.md) |                                   |               | Bill linkage                 |
| Debt Charge ID     | Ref → [Debt Charges](debt-charges.md) |                                   |               | Interest/Fee linkage         |
| TransLink          | Ref → [Transaction Links](transaction-links.md) |                         |               | Bill allocation source       |
| PmtAllocID         | Ref → [PaymentAllocations](payment-allocations.md) |                      |               | Debt allocation source       |
| StatementID        | Ref → [Statements](statements.md)    |                                    |               | Statement linkage            |
| Src Row ID         | Text                                 |                                    |               | Originating row              |
| Notes              | Text                                 |                                    |               | Optional                     |
| Label2             | Text (VC)                            | [Formula](#label2-formula)         |               | Human-readable label         |
| Installment Schedule ID | Ref → [Installment Schedule](payments.md) |                          |               | link to installment sched    |
| Allocation Role    | Enum                                 |                                    |               | Scheduled, Extra, etc        |
| Allocation Type    | Enum                                 |                                    |               | Principal / Interest / Fee   |
| Counts Towards Installment | Yes/No                       |                                    |               | To assist with installment   |
| IsCashMovement     | Yes/No (VC)                          | [Formula](#movement-formula)       |               | To assist with ledger math   |
| Bill/Debt          | Text (VC)                            | [Formula](#bill-debt-formula)      |               | To assist with ledger view   |
| Ledger Group Key   | Text (VC)                            | [Formula](#ledger-group-formula)   |               | To assist with ledger view   |
| Ledger Group Sort  | Text (VC)                            | [Formula](#ledger-sort-formula)    |               | To assist with ledger view   |
| Matched PmtAllocID | Text (VC)                            | [Formula](#matched-alloc-formula)  |               | To assist with bot process   |


</details>

---

## key-virtual-columns

<details>
<summary>View Virtual Columns</summary>

| Column        | Type       | Formula                                | Notes            |
| ------------- | ---------- | -------------------------------------- | ---------------- |
| Signed Amount | Price (VC) | [View Formula](#signed-amount-formula) | Core ledger math |
| Label2        | Text (VC)  | [View Formula](#label2-formula)        | Display label    |

</details>

---

## formulas
<details>
<summary>View Formulas</summary>



### signed-amount-formula

<details>
<summary>View Formula</summary>

```appsheet
[Amount]
*
IFS(
  IN([Type], {"Bill Payment", "Debt Payment", "Debt Paid Off", "Principal Applied"}), -1,

  [Type] = "Adjustment",
    IFS(
      IN([Charge Category], {"Fee", "Late Fee", "NSF Fee", "Penalty", "Interest","Purchase"}), 1,
      IN([Charge Category], {"Credit", "Refund", "Waiver", "Correction"}), -1,
      TRUE, -1
    ),

  [Type] = "Debt Charge",
    IF(
      [Debt Charge ID].[Is Reversal?] = TRUE,
      -1,
      1
    ),

  IN([Type], {"Bill Payment Reversal", "Debt Payment Reversal", "Principal Reversal", "Charge"}), 1,

  TRUE, 1
)
```

</details>

---

### label2-formula

<details>
<summary>View Formula</summary>

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

### label-formula
<details>
<summary>View Formula</summary>
```appsheet
CONCATENATE(
  [Type],
" - ",
LEFT([Src Row ID],5),
" | $",
[Signed Amount]
)
```
</details>

---

### movement-formula
<details>
<summary>View Formula</summary>

```appsheet
IN(
  [Type],
  LIST(
    "Bill Payment",
    "Debt Payment",
    "Bill Payment Reversal",
    "Debt Payment Reversal"
  )
)
```
</details>

---

### bill-debt-formula
<details>
<summary>View Formula</summary>

```appsheet
IF(
  ISNOTBLANK([Bill ID]),
  Bill,
  Debt
)
```
</details>

---

### ledger-group-formula
<details>
<summary>View Formula</summary>

```appsheet
IFS(
  ISNOTBLANK([Bill ID]),
    "Bill: " & [Bill ID].[Bill Name],

  ISNOTBLANK([Debt ID]),
    "Debt: " & [Debt ID].[Debt Name],

  TRUE,
    "Unassigned"
)
```
</details>

---

### ledger-sort-formula
<details>
<summary>View Formula</summary>

```appsheet
IFS(
  ISNOTBLANK([Bill ID]),
    [Bill ID].[Bill Name],

  ISNOTBLANK([Debt ID]),
    [Debt ID].[Debt Name],

  TRUE,
    "Unassigned"
)
```
</details>

---

### matched-alloc-formula
<details>
<summary>View Formula</summary>

```appsheet
IF(
  ISNOTBLANK([_THISROW].[Debt Charge ID]),
  ANY(
    SELECT(
      PaymentAllocations[PaymentAllocationID],
      AND(
        [DebtID] = [_THISROW].[Debt ID],
        [Debt Charge ID] = [_THISROW].[Debt Charge ID],
        [Amount] = ABS([_THISROW].[Amount])
      )
    )
  ),
  ANY(
    SELECT(
      PaymentAllocations[PaymentAllocationID],
      AND(
        [DebtID] = [_THISROW].[Debt ID],
        [Payment ID] = [_THISROW].[Payment ID],
        [Amount] = ABS([_THISROW].[Amount])
      )
    )
  )
)
```
</details>

---
</details>

---
---

## event-types

<details>
<summary>View Event Types</summary>

### charges (positive)

| Type        | Description    |
| ----------- | -------------- |
| Charge      | Bill charge    |
| Debt Charge | Interest / Fee |

---

### payments (negative)

| Type              | Description          |
| ----------------- | -------------------- |
| Bill Payment      | Bill payment         |
| Debt Payment      | Interest/Fee payment |
| Principal Applied | Principal reduction  |

---

### reversals (positive)

| Type                  | Description       |
| --------------------- | ----------------- |
| Bill Payment Reversal | Undo bill payment |
| Debt Payment Reversal | Undo interest/fee |
| Principal Reversal    | Undo principal    |

---

### adjustments

| Type       | Description       |
| ---------- | ----------------- |
| Adjustment | Manual correction |

</details>

---
---

## actions

<details>
<summary>View Actions</summary>

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
---

## bots

<details>
<summary>View Bot Interactions</summary>

### ledgerbot

Trigger:

* On Transaction Links add

Creates:

* Bill Payment ledger rows

---

### pmt-allocation-bot

Trigger:

* On PaymentAllocations add

Creates:

* Principal Applied ledger
* Debt Payment ledger

---

### debt-charge-bot

Trigger:

* On Debt Charge add

Creates:

* Positive Debt Charge ledger

---

### reversal-flow

* Reversal Payments create mirrored ledger rows
* Must invert Signed Amount behavior

</details>

---
---

## relationships

<details>
<summary>View Relationships</summary>

```text
Bills → Bill Charges → Transaction Links → Ledger
Payments → Transaction Links → Ledger
Payments → PaymentAllocations → Ledger
Debts → Debt Charges → Ledger
Statements → PaymentAllocations → Ledger
```

</details>

---
---

## rules

<details>
<summary>View Critical Rules</summary>

### no-duplicate-financial-impact

Each real-world event should produce exactly one ledger effect.

---

### allocation-overrides-full-payment

If allocations exist:

* DO NOT apply full payment as balance change

---

### affects-balance-must-be-correct

Incorrect usage breaks all balances.

---

### ledger-must-be-complete

Missing rows = broken system

</details>

---
---

## known-issues

<details>
<summary>View Known Issues</summary>

| Issue                 | Cause                     |
| --------------------- | ------------------------- |
| Balance mismatch      | Missing or duplicate rows |
| Principal incorrect   | Double-counting           |
| Interest not clearing | Missing allocation        |
| Rounding errors       | Precision issues          |

</details>

---
---

## debug-strategy

<details>
<summary>View Debug Workflow</summary>

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
---

## repair-notes

<details>
<summary>View Repair Strategies</summary>

### disable-legacy-full-payment-ledger

```text
Affects Balance? = FALSE
```

---

### backfill-missing-pmtallocid

Ensure all allocation-driven ledger rows are linked.

---

### normalize-classification

Ensure:

* Charge Category is set correctly
* Allocation Type is set correctly

---

### enforce-1-to-1-allocation-ledger

Each PaymentAllocation should create exactly one ledger row.

</details>

---
---

## related-docs

* `docs/data-model/debts.md`
* `docs/data-model/payments.md`
* `docs/workflows/payment-allocation-flow.md`
* `docs/workflows/loan-statement-flow.md`
