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

## balance-contracts

<details>
<summary>View Balance Contracts</summary>

### ledger-as-ssot

`Ledger` is the source of truth for balance math.

Source tables such as `Bill Charges`, `Debt Charges`, `Payments`, `Transaction Links`, `PaymentAllocations`, and `Adjustments` may describe the event, allocation, workflow state, or source row, but they should not be the final authority for balance columns.

Any balance-affecting event should be represented by a ledger row, and balance columns should derive from:

```text
SUM(Ledger[Signed Amount])
```

---

### bill-charge-balance-contract

A Bill Charge balance is calculated from ledger rows linked to that specific Bill Charge:

```text
SUM(
  Ledger[Signed Amount]
  WHERE Ledger[Bill Charge ID] = BillCharges[Row ID]
)
```

The UI-facing value in `Bill Charges[Remaining Amount]` should clamp that result to zero.

Recommended `Bill Charges[Remaining Amount]` AppSheet formula:

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

Use this `IF()` clamp instead of `MAX(LIST(...))` because AppSheet may treat `0.0` as Decimal and `SUM(Ledger[Signed Amount])` as Price, causing a mismatched list type error.

---

### bill-charge-ledger-row-requirements

Every Bill Charge should have a complete event trail:

1. Original `Charge` ledger row when the Bill Charge is created.
2. `Bill Payment` ledger row for every amount applied to that Bill Charge.
3. `Adjustment` ledger row for every credit, fee, refund, waiver, correction, or manual balance change tied to that Bill Charge.
4. `Bill Payment Reversal` ledger row when a previously applied payment is returned, reversed, or otherwise undone.

Do not switch a historical Bill Charge fully to Ledger-driven balance math until it has the required positive original `Charge` ledger row. Otherwise, old charges with only payment or adjustment rows may incorrectly calculate as zero after clamping.

</details>

---
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

## sign-convention

<details>
<summary>View Required Sign Convention</summary>

`Signed Amount` should represent the effect on the balance, not whether money physically moved.

| Event / Ledger Type | Signed Amount Direction | Balance Effect |
| ------------------- | ----------------------- | -------------- |
| `Charge` | Positive | Increases amount owed |
| `Debt Charge` | Positive | Increases amount owed |
| `Interest Charge` | Positive | Increases amount owed |
| `Bill Payment` | Negative | Reduces amount owed |
| `Debt Payment` | Negative | Reduces amount owed |
| `Principal Applied` | Negative | Reduces principal owed |
| `Adjustment` / fee | Positive | Increases amount owed |
| `Adjustment` / purchase | Positive | Increases amount owed |
| `Adjustment` / credit | Negative | Reduces amount owed |
| `Adjustment` / refund | Negative | Reduces amount owed unless intentionally modeled differently |
| `Adjustment` / waiver | Negative | Reduces amount owed |
| `Bill Payment Reversal` | Positive | Restores amount owed |
| `Debt Payment Reversal` | Positive | Restores amount owed |
| `Principal Reversal` | Positive | Restores principal owed |
| `Debt Paid Off` | Negative | Clears remaining owed amount |

---

### adjustment-sign-rule

If `Adjustments[Signed Amount]` already contains the correct positive or negative value, ledger rows created from adjustments should copy that value rather than recalculating the sign independently.

This prevents disagreement between the adjustment source row and the ledger row.

---

### signed-amount-pattern-reference

The current `Signed Amount` formula below is the active implementation. Conceptually, it should match this pattern:

```text
Charges and fees are positive.
Payments, credits, waivers, and reductions are negative.
Reversals invert the original event.
```

</details>

---
---

## event-types

<details>
<summary>View Event Types</summary>

### charges-positive

| Type | Description | Signed Amount |
| ---- | ----------- | ------------- |
| Charge | Bill charge | Positive |
| Debt Charge | Interest / Fee | Positive unless `[Debt Charge ID].[Is Reversal?] = TRUE` |
| Interest Charge | Interest event, if used separately from `Debt Charge` | Positive |

---

### payments-negative

| Type | Description | Signed Amount |
| ---- | ----------- | ------------- |
| Bill Payment | Bill payment applied to a Bill Charge | Negative |
| Debt Payment | Debt payment applied to interest or fees | Negative |
| Principal Applied | Principal reduction | Negative |
| Debt Paid Off | Payoff event | Negative |

---

### reversals-positive

| Type | Description | Signed Amount |
| ---- | ----------- | ------------- |
| Bill Payment Reversal | Undo bill payment / returned payment | Positive |
| Debt Payment Reversal | Undo debt payment | Positive |
| Principal Reversal | Undo principal reduction | Positive |

---

### adjustments-variable

| Type | Description | Signed Amount |
| ---- | ----------- | ------------- |
| Adjustment | Manual correction | Based on `Charge Category` or copied from `Adjustments[Signed Amount]` |

Adjustment category convention:

| Charge Category | Direction |
| --------------- | --------- |
| Fee | Positive |
| Late Fee | Positive |
| NSF Fee | Positive |
| Penalty | Positive |
| Interest | Positive |
| Purchase | Positive |
| Credit | Negative |
| Refund | Negative unless intentionally modeled as money returned after overpayment |
| Waiver | Negative |
| Correction | Negative by default unless the correction increases the balance |

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
* Ensure every balance-affecting source row creates exactly one intended ledger impact
* Ensure Bill Charge-related ledger rows include `Bill Charge ID`

Bill Charge-specific actions should support:

* Add Original Bill Charge Ledger Row
* Add Bill Payment Ledger Row
* Add Bill Adjustment Ledger Row
* Add Bill Payment Reversal Ledger Row

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

* `Bill Payment` ledger rows
* Negative `Signed Amount` entries linked to the applied `Bill Charge ID`

Rule:

* LedgerBot should not create display-only audit rows. It must create the balance-affecting row used by Bill Charge remaining amount calculations.

---

### pmt-allocation-bot

Trigger:

* On PaymentAllocations add

Creates:

* Principal Applied ledger
* Debt Payment ledger

---

### bill-charge-bot

Trigger:

* On Bill Charge add

Creates:

* Positive `Charge` ledger row linked to the new `Bill Charge ID`

This row is required before `Bill Charges[Remaining Amount]` can safely use Ledger as SSOT.

---

### adjustment-bot

Trigger:

* On Adjustment add or update when the adjustment affects a Bill Charge or Debt Charge

Creates:

* `Adjustment` ledger row
* Positive or negative `Signed Amount` based on the adjustment sign/category
* Correct `Bill Charge ID` or `Debt Charge ID` linkage

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
Bills → Bill Charges → Ledger
Bills → Bill Charges → Transaction Links → Ledger
Payments → Transaction Links → Ledger
Adjustments → Ledger
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

---

### ledger-drives-balances

Balance columns should read from Ledger wherever possible.

Examples:

* `Bill Charges[Remaining Amount]`
* debt principal / interest balances
* statement balance reconciliation
* dashboard totals

---

### do-not-use-ledger-for-display-only

Ledger must not be treated as a passive audit log.

If a balance changes, the ledger must contain the event that caused the change.

---

### bill-charge-ledger-completeness

For Bill Charge balance math to be reliable, each Bill Charge needs:

* one original positive `Charge` ledger row
* zero or more negative `Bill Payment` rows
* zero or more positive/negative `Adjustment` rows
* zero or more positive `Bill Payment Reversal` rows

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

## reconciliation-checks

<details>
<summary>View Reconciliation Checks</summary>

### bill-charges-missing-original-charge-ledger-row

Use this as a Bill Charges slice/filter to find charges that cannot safely use Ledger-driven remaining balance yet.

```appsheet
COUNT(
  SELECT(
    Ledger[LedgerID],
    AND(
      [Bill Charge ID] = [_THISROW].[Row ID],
      [Type] = "Charge"
    )
  )
) = 0
```

---

### bill-charges-with-multiple-original-charge-ledger-rows

Use this to find duplicated original charge impacts.

```appsheet
COUNT(
  SELECT(
    Ledger[LedgerID],
    AND(
      [Bill Charge ID] = [_THISROW].[Row ID],
      [Type] = "Charge"
    )
  )
) > 1
```

---

### ledger-rows-missing-bill-charge-link

Use this as an admin slice for bill-related ledger rows that should affect a Bill Charge but are not linked correctly.

```appsheet
AND(
  IN(
    [Type],
    LIST(
      "Charge",
      "Bill Payment",
      "Adjustment",
      "Bill Payment Reversal"
    )
  ),
  ISBLANK([Bill Charge ID]),
  ISNOTBLANK([Bill ID])
)
```

---

### bill-charge-ledger-total

Use this as a diagnostic virtual column or temporary check while migrating to Ledger SSOT.

```appsheet
SUM(
  SELECT(
    Ledger[Signed Amount],
    AND(
      [Bill Charge ID] = [_THISROW].[Row ID],
      ISNOTBLANK([Signed Amount])
    )
  )
)
```

---

### bill-charge-ledger-remaining-clamped

This should match the production `Bill Charges[Remaining Amount]` formula after migration.

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

</details>

---
---

## repair-notes

<details>
<summary>View Repair Strategies</summary>

### backfill-original-bill-charge-ledger-rows

Before switching old Bill Charges to Ledger-driven remaining balances, create one positive `Charge` ledger row for each historical Bill Charge that does not already have one.

Required fields:

* `Type` = `Charge`
* `Amount` = original Bill Charge amount
* `Bill ID` = source bill
* `Bill Charge ID` = source Bill Charge
* `Date` = Bill Charge date/cycle date
* `Src Row ID` = Bill Charge key

---

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

* `docs/data-model/bill-charges.md`
* `docs/data-model/debts.md`
* `docs/data-model/payments.md`
* `docs/workflows/payment-allocation-flow.md`
* `docs/workflows/loan-statement-flow.md`
* `docs/decisions/ADR-0002-ledger-ssot-for-bill-charge-balances.md`
