# 📋 Bill Charges.md

> Last Updated: 2026-06-17
>
> Related Docs:
>
> - [ledger.md](ledger.md)
> - [bills.md](bills.md)
> - [payments.md](payments.md)
> - [transaction-links.md](transaction-links.md)
> - [adjustments.md](adjustments.md)
> - [handling-returned-payments.md](../processes/handling-returned-payments.md)

---

# 📌 Overview

## Purpose

`Bill Charges` represents a single bill obligation for a specific billing cycle.

Each row identifies the charge amount, cycle metadata, parent bill, and related payment/allocation activity. The table should describe the charge, but it should not be the source of truth for the remaining balance once the Ledger-as-source-of-truth model is active.

The remaining balance for a Bill Charge should be derived from `Ledger[Signed Amount]` entries where `Ledger[Bill Charge ID]` points to the current Bill Charge row.

---

# 🧠 Design Philosophy

<details open>
<summary><strong>Expand Design Notes</strong></summary>

`Bill Charges` exists to turn recurring or one-off bills into concrete payable obligations.

A `Bills` row describes the recurring bill/account. A `Bill Charges` row describes the specific cycle charge that must be paid, reduced, credited, reversed, or closed.

This table SHOULD store:

- The parent bill reference
- The original charge amount
- Cycle metadata such as cycle label, start date, and end date
- A human-readable charge type/category
- References to related adjustments, payments, transaction links, and ledger rows
- A virtual remaining amount derived from `Ledger`

This table SHOULD NOT store:

- Final financial truth independent of Ledger
- Direct payment math from `Transaction Links`
- Direct adjustment math from `Adjustments`
- Manual balance overrides except as repair/admin fields, if intentionally added later

The Ledger owns the balance. `Bill Charges` owns charge identity and cycle context.

```text
If it affects the remaining amount of a Bill Charge, it must exist in Ledger.
```

</details>

---

# 🗂️ Table Relationships

<details open>
<summary><strong>Expand Relationship Diagram</strong></summary>

```text
Bills
  ↓
Bill Charges
  ↓
Transaction Links
  ↓
Ledger

Payments
  ↓
Transaction Links
  ↓
Ledger

Adjustments
  ↓
Ledger

Returned Payments / Reversals
  ↓
Ledger
  ↓
Bill Charges[Remaining Amount]
```

Balance flow:

```text
Bill Charges[Remaining Amount]
  = SUM(
      Ledger[Signed Amount]
      where Ledger[Bill Charge ID] = this Bill Charge
    )
```

</details>

---

# 🔑 Core Columns

<details open>
<summary><strong>Expand Core Columns</strong></summary>

| Column | Type | Purpose |
|---|---|---|
| Row ID | Text | Primary key for the Bill Charge row |
| Bill ID | Ref → Bills | Parent bill/account this charge belongs to |
| Date | Date | Charge date or cycle anchor date |
| Amount | Price | Original amount of the charge before payments, credits, or reversals |
| ChargeType | Enum/Text | Classifies the charge, such as monthly charge, fee, correction, or other cycle event |
| CycleLabel | Text | Human-readable billing cycle label |
| CycleStart | Date | Start date of the charge cycle |
| CycleEnd | Date | End date of the charge cycle |
| Remaining Amount | Price VC | Ledger-derived remaining amount still owed for this charge |

</details>

---

# 📚 ALL Columns

<details open>
<summary><strong>Expand Full Column Reference</strong></summary>

## Row ID

### Column Name

```text
Row ID
```

### Type

```text
Text
```

### Purpose

Primary key for the `Bill Charges` table.

### Formula

```appsheet
UNIQUEID()
```

### Notes

- Usually set as the Initial Value, not an App Formula.
- Referenced by `Ledger[Bill Charge ID]`.
- Referenced by payment allocation/transaction-link flows.

---

## Bill ID

### Column Name

```text
Bill ID
```

### Type

```text
Ref → Bills
```

### Purpose

Connects the charge to its parent bill.

### Formula

```appsheet

```

### Notes

- Required for normal bill charge rows.
- Used for rollups from Bills to Bill Charges.
- Should also be copied to related Ledger rows when applicable.

---

## Date

### Column Name

```text
Date
```

### Type

```text
Date
```

### Purpose

Represents the charge date or billing-cycle anchor date.

### Formula

```appsheet

```

### Notes

- May align with bill due date, statement date, or generated cycle date depending on the bill workflow.
- Should be consistent with `CycleStart`, `CycleEnd`, and `CycleLabel`.

---

## Amount

### Column Name

```text
Amount
```

### Type

```text
Price
```

### Purpose

Stores the original charge amount for this bill cycle.

### Formula

```appsheet

```

### Notes

- This is descriptive/input data for the original obligation.
- Under Ledger SSOT, this column is not the final balance source.
- The original charge amount should be represented by a positive `Ledger` row with `Type = "Charge"`.

---

## ChargeType

### Column Name

```text
ChargeType
```

### Type

```text
Enum/Text
```

### Purpose

Classifies the kind of charge.

### Formula

```appsheet

```

### Notes

- Useful for reporting and debugging.
- Examples may include regular cycle charge, fee, manual charge, correction, or backfill.
- Ledger impact is controlled by Ledger row type/sign, not by this field directly.

---

## CycleLabel

### Column Name

```text
CycleLabel
```

### Type

```text
Text
```

### Purpose

Human-readable label for the billing cycle.

### Formula

```appsheet

```

### Notes

- Recommended format: `MMM YYYY` or another consistent cycle label.
- Helps group charges, payments, and debug logs by period.

---

## CycleStart

### Column Name

```text
CycleStart
```

### Type

```text
Date
```

### Purpose

Start date of the billing cycle represented by this charge.

### Formula

```appsheet

```

### Notes

- Used for cycle filtering and historical reconciliation.
- Should not be used alone as a balance source.

---

## CycleEnd

### Column Name

```text
CycleEnd
```

### Type

```text
Date
```

### Purpose

End date of the billing cycle represented by this charge.

### Formula

```appsheet

```

### Notes

- Used for cycle filtering and payment repair logic.
- Should be consistent with the parent bill frequency.

---

## Remaining Amount

### Column Name

```text
Remaining Amount
```

### Type

```text
Price Virtual Column
```

### Purpose

Calculates the amount still owed for this Bill Charge from Ledger entries tied to this row.

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

### Notes

- This is the final Ledger SSOT formula.
- It avoids `MAX(LIST(0.0, PriceValue))` because AppSheet may treat `0.0` as Decimal and `SUM(Ledger[Signed Amount])` as Price, causing a mismatched list type error.
- Do not directly subtract `Transaction Links[Amount Applied]` in this formula after Ledger SSOT cutover.
- Do not directly add `Adjustments[Signed Amount]` in this formula after Ledger SSOT cutover.

---

## Related Adjustments

### Column Name

```text
Related Adjustments
```

### Type

```text
List / Reverse Ref
```

### Purpose

Shows adjustment rows associated with this Bill Charge.

### Formula

```appsheet
REF_ROWS("Adjustments", "Bill Charge ID")
```

### Notes

- Useful for navigation and audit detail.
- Adjustment rows are not the balance source once Ledger SSOT is active.
- Adjustment financial impact must be represented in Ledger.

---

## Related Payments

### Column Name

```text
Related Payments
```

### Type

```text
List / Reverse Ref or Virtual Column
```

### Purpose

Shows payments associated with this Bill Charge.

### Formula

```appsheet

```

### Notes

- Depending on the app structure, this may be direct or may be inferred through Transaction Links.
- Payment rows are not the balance source once Ledger SSOT is active.
- Payment financial impact must be represented in Ledger.

---

## Related Transaction Links

### Column Name

```text
Related Transaction Links
```

### Type

```text
List / Reverse Ref
```

### Purpose

Shows transaction allocation rows that connect payments to this Bill Charge.

### Formula

```appsheet
REF_ROWS("Transaction Links", "Bill Charge ID")
```

### Notes

- Transaction Links remain important allocation detail.
- They should explain how a payment was applied.
- They should not be summed directly by `Remaining Amount` after Ledger SSOT cutover.
- Ledger rows created from Transaction Links should carry the actual balance impact.

---

## Related Ledger Rows

### Column Name

```text
Related Ledger Rows
```

### Type

```text
List / Reverse Ref
```

### Purpose

Shows Ledger rows tied to this Bill Charge.

### Formula

```appsheet
REF_ROWS("Ledger", "Bill Charge ID")
```

### Notes

- Recommended if not already present.
- This is the preferred audit trail for remaining amount math.
- Useful for troubleshooting missing charge, payment, adjustment, or reversal entries.

---

</details>

---

# ⚙️ Virtual Columns

<details>
<summary><strong>Expand Virtual Columns</strong></summary>

## Remaining Amount

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

Purpose:

- Sums all Ledger activity tied to the current Bill Charge.
- Clamps negative balances to zero.
- Treats Ledger as the only financial source of truth.

## Related Ledger Rows

```appsheet
REF_ROWS("Ledger", "Bill Charge ID")
```

Purpose:

- Provides an audit list of all Ledger entries that affect this Bill Charge.
- Recommended for debugging and reconciliation.

</details>

---

# 🧮 Formula Reference

<details>
<summary><strong>Expand Formula Reference</strong></summary>

## Ledger SSOT Remaining Amount

Use this after all historical Bill Charges have original positive `Charge` ledger rows.

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

## Transitional Remaining Amount

Use only during migration if historical original charge rows have not been backfilled yet.

```appsheet
IF(
  [Amount]
  + SUM(
    SELECT(
      Ledger[Signed Amount],
      AND(
        [Bill Charge ID] = [_THISROW].[Row ID],
        [Type] <> "Charge",
        ISNOTBLANK([Signed Amount])
      )
    )
  ) > 0,
  [Amount]
  + SUM(
    SELECT(
      Ledger[Signed Amount],
      AND(
        [Bill Charge ID] = [_THISROW].[Row ID],
        [Type] <> "Charge",
        ISNOTBLANK([Signed Amount])
      )
    )
  ),
  0
)
```

This is not the final SSOT formula because it still uses `Bill Charges[Amount]` as part of the balance calculation.

## Legacy Remaining Amount Diagnostic

Use only as a temporary reconciliation check.

```appsheet
IF(
  [Amount]
  + SUM([Related Adjustments][Signed Amount])
  - SUM([Related Transaction Links][Amount Applied])
  > 0,
  [Amount]
  + SUM([Related Adjustments][Signed Amount])
  - SUM([Related Transaction Links][Amount Applied]),
  0
)
```

## Ledger Remaining Amount Diagnostic

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

## Mismatch Flag

```appsheet
ABS(
  [Legacy Remaining Amount Check]
  - [Ledger Remaining Amount Check]
) > 0.01
```

</details>

---

# 🔄 Automation Dependencies

<details open>
<summary><strong>Expand Automation Dependencies</strong></summary>

## Bots

- Bill Charge creation bot/process
- LedgerBot
- Payment / Transaction Link bot
- Adjustment bot
- Returned Payment / Reversal bot
- Bill Refresh bot

---

## Actions

- Create original Bill Charge ledger row
- Create Bill Payment ledger row from Transaction Link
- Create Bill Adjustment ledger row
- Create Bill Payment Reversal ledger row
- Refresh Bill status
- Recalculate or sync related bill fields
- Repair missing Ledger references

---

## Processes

- Bill charge generation
- Bill payment allocation
- Bill charge catch-up/backfill allocation
- Returned payment handling
- Ledger reconciliation
- Historical charge ledger backfill

</details>

---

# 🧪 Testing Checklist

<details open>
<summary><strong>Expand Testing Checklist</strong></summary>

- [ ] New Bill Charge creates one positive `Ledger` row with `Type = "Charge"`
- [ ] Bill payment creates a negative `Ledger` row tied to the same `Bill Charge ID`
- [ ] Credit adjustment creates a negative `Ledger` row tied to the same `Bill Charge ID`
- [ ] Fee/debit adjustment creates a positive `Ledger` row tied to the same `Bill Charge ID`
- [ ] Returned payment creates a positive reversal `Ledger` row tied to the same `Bill Charge ID`
- [ ] `Remaining Amount` equals the sum of related signed Ledger rows when positive
- [ ] `Remaining Amount` clamps to `0` when Ledger sum is zero or negative
- [ ] No circular references are introduced between Bill Charges, Transaction Links, Payments, and Ledger
- [ ] Historical Bill Charges have original positive `Charge` ledger rows before final cutover
- [ ] Legacy remaining amount and Ledger remaining amount reconcile within `$0.01`

</details>

---

# 🐞 Known Issues / Edge Cases

<details>
<summary><strong>Expand Known Issues</strong></summary>

- `MAX(LIST(0.0, SUM(Ledger[Signed Amount])))` can fail in AppSheet if `0.0` is treated as Decimal and the Ledger sum is treated as Price.
- Missing original `Charge` ledger rows will cause Ledger SSOT remaining amount to be understated.
- Duplicate original `Charge` ledger rows will overstate the remaining amount.
- Transaction Links without corresponding Ledger rows will not affect balance under SSOT.
- Adjustments without corresponding Ledger rows will not affect balance under SSOT.
- Returned payments must create reversal Ledger rows; they should not directly edit the Bill Charge balance.
- Overpayments may create a negative Ledger sum, but `Remaining Amount` should display `0` unless a separate credit/prepayment tracking model is added.

</details>

---

# 🚀 Future Improvements

<details>
<summary><strong>Expand Future Improvements</strong></summary>

- [ ] Add `Related Ledger Rows` reverse reference if it does not already exist
- [ ] Add admin-only diagnostic columns for Ledger vs legacy balance comparison
- [ ] Add integrity validator for missing original `Charge` ledger rows
- [ ] Add duplicate `Charge` ledger row detector
- [ ] Add explicit `Ledger[Affects Balance?]` physical or virtual flag if void/superseded Ledger rows become common
- [ ] Add repair action to create missing Bill Charge ledger rows
- [ ] Add dashboard slice for Bill Charges where Ledger and legacy remaining amounts do not match

</details>

---

# 🔍 Cross References

## Related Tables

- Bills
- Payments
- Transaction Links
- Ledger
- Adjustments
- Debug Log

---

## Related Systems

- Ledger System
- Bill Payment Allocation System
- Returned Payment Handling
- Bill Refresh / Status System
- Ledger Reconciliation

---

# 📚 Change History

<details open>
<summary><strong>Expand Change History</strong></summary>

| Date | Change |
|---|---|
| 2026-05-08 | Initial table documentation template created |
| 2026-06-17 | Recreated `bill-charges.md` using default CashSnap table template |
| 2026-06-17 | Updated `Remaining Amount` to use Ledger SSOT formula with AppSheet-safe `IF()` clamp |
| 2026-06-17 | Added migration, reconciliation, and returned payment/reversal notes |

</details>
