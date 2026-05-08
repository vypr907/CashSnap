# 🚗 Auto Loan Repair Status

> Last Updated: 2026-05-08
>
> Related Docs:
>
> - [loan-statement-process-repair-plan.md](./loan-statement-process-repair-plan.md)
> - [debt-ledger-ssot-formulas.md](../formulas/debt-ledger-ssot-formulas.md)
> - [pmt-allocation-bot.md](../bots/pmt-allocation-bot.md)

---

# 📌 Overview

This document tracks the current repair status for the Auto Loan debt using:

```text
Debt Type: Loan
Model: Loan (Statement-Based)
```

The repair goal is to align the debt with the CashSnap Ledger-as-source-of-truth model.

---

# ✅ Current Snapshot

<details open>
<summary><strong>Expand Current Values</strong></summary>

| Field | Current Value | Status |
|---|---:|---|
| Starting Balance | 20524.00 | Source baseline |
| Paid So Far | 13923.49 | Legacy/general cash paid value |
| Remaining Balance | 12974.00 | Still under review |
| Principal Paid | 10272.16 | Looks correct |
| Principal_Balance | 10251.84 | Correct after refresh |
| Interest_Balance | 2722.96 | Needs interest payment repair |
| Interest Charged | 2845.41 | Looks populated |
| Interest Paid | 122.45 | Too low / incomplete |
| Ledger Balance | 10251.84 | Matches Principal_Balance |

</details>

---

# 🧠 Interpretation

<details open>
<summary><strong>Expand Analysis</strong></summary>

The principal side is now working:

```text
20524.00 - 10272.16 = 10251.84
```

This matches:

```text
Ledger Balance = 10251.84
```

The remaining problem is interest-side repair:

```text
2845.41 - 122.45 = 2722.96
```

This means `Interest Paid` is likely not detecting all interest payment ledgers.

Likely causes:

- older interest payment Ledger rows missing `Allocation Type = Interest`
- older rows missing `PmtAllocID`
- older rows missing `Debt Charge ID`
- older rows using a Ledger `Type` not included in the formula
- older rows have `Charge Category = Interest` but no `Allocation Type`

</details>

---

# 🔧 Next Repair Steps

<details open>
<summary><strong>Expand Next Steps</strong></summary>

## Step 1 — Inspect Interest Payment Ledger Rows

Filter Ledger rows where:

```appsheet
AND(
  [Debt ID] = "<<Auto Loan Debt ID>>",
  OR(
    [Charge Category] = "Interest",
    [Allocation Type] = "Interest",
    AND(
      ISNOTBLANK([Debt Charge ID]),
      [Debt Charge ID].[ChargeType] = "Interest"
    )
  )
)
```

Check whether each row has:

- `Allocation Type`
- `Charge Category`
- `Debt Charge ID`
- `PmtAllocID`
- `StatementID`
- correct `Type`

---

## Step 2 — Run Ledger Repair Action

Run the Ledger repair/enrichment action on Auto Loan ledger rows to fill:

- `Charge Category`
- `PmtAllocID`
- `Allocation Type`
- `StatementID`

---

## Step 3 — Recheck Interest Paid

After repair, verify whether `Interest Paid` increases from:

```text
122.45
```

toward the expected total interest paid.

---

## Step 4 — Recheck Remaining Balance

Once Interest Paid is repaired:

```text
Remaining Balance = Principal_Balance + Interest_Balance
```

should become accurate.

</details>

---

# ⚠️ Known Issue

<details>
<summary><strong>Physical Formula Refresh</strong></summary>

Some debt fields are physical columns with formulas. After formula changes or ledger repairs, the Debt row may need to be refreshed or updated so physical formula columns recalculate.

</details>
