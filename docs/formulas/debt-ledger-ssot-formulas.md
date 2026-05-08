# 🧮 Debt Ledger SSOT Formulas

> Last Updated: 2026-05-08
>
> Related Docs:
>
> - [auto-loan-repair-status.md](../processes/auto-loan-repair-status.md)
> - [loan-statement-process-repair-plan.md](../processes/loan-statement-process-repair-plan.md)
> - [ledger.md](../tables/ledger.md)

---

# 📌 Overview

These formulas align loan balance calculations with the CashSnap Ledger-as-source-of-truth model.

Recommended ownership:

| Data | Source |
|---|---|
| Principal paid | Ledger |
| Interest charged | Ledger |
| Interest paid | Ledger |
| Principal balance | Starting Balance - Principal Paid |
| Interest balance | Interest Charged - Interest Paid |
| Remaining balance | Principal Balance + Interest Balance |

---

# 🧾 Principal Paid

<details open>
<summary><strong>Expand Formula</strong></summary>

```appsheet
ABS(
  SUM(
    SELECT(
      Ledger[Signed Amount],
      AND(
        [Debt ID] = [_THISROW].[Row ID],
        [Allocation Type] = "Principal",
        [Type] = "Principal Applied"
      )
    )
  )
)
```

</details>

---

# 🧾 Principal_Balance

<details open>
<summary><strong>Expand Formula</strong></summary>

```appsheet
IF(
  IN(
    [Model],
    LIST(
      "Loan (Transaction-Based)",
      "Loan (Statement-Based)"
    )
  ),
  [Starting Balance] - [Principal Paid],
  [Starting Balance]
)
```

</details>

---

# 🧾 Interest Charged

<details open>
<summary><strong>Expand Formula</strong></summary>

```appsheet
SUM(
  SELECT(
    Ledger[Signed Amount],
    AND(
      [Debt ID] = [_THISROW].[Row ID],
      [Type] = "Debt Charge",
      [Charge Category] = "Interest"
    )
  )
)
```

</details>

---

# 🧾 Interest Paid

<details open>
<summary><strong>Clean Formula</strong></summary>

Use after ledger rows are repaired and consistently populated.

```appsheet
ABS(
  SUM(
    SELECT(
      Ledger[Signed Amount],
      AND(
        [Debt ID] = [_THISROW].[Row ID],
        [Allocation Type] = "Interest",
        IN(
          [Type],
          LIST(
            "Debt Payment",
            "Interest Payment"
          )
        )
      )
    )
  )
)
```

</details>

<details>
<summary><strong>Temporary Repair-Friendly Formula</strong></summary>

Use while older ledger rows are missing `Allocation Type`.

```appsheet
ABS(
  SUM(
    SELECT(
      Ledger[Signed Amount],
      AND(
        [Debt ID] = [_THISROW].[Row ID],
        OR(
          [Allocation Type] = "Interest",
          [Charge Category] = "Interest",
          AND(
            ISNOTBLANK([Debt Charge ID]),
            [Debt Charge ID].[ChargeType] = "Interest"
          )
        ),
        IN(
          [Type],
          LIST(
            "Debt Payment",
            "Interest Payment"
          )
        )
      )
    )
  )
)
```

</details>

---

# 🧾 Interest_Balance

<details open>
<summary><strong>Expand Formula</strong></summary>

```appsheet
IF(
  IN(
    [Model],
    LIST(
      "Loan (Transaction-Based)",
      "Loan (Statement-Based)"
    )
  ),
  [Interest Charged] - [Interest Paid],
  0.00
)
```

</details>

---

# 🧾 Remaining Balance

<details open>
<summary><strong>Relevant Loan Branch</strong></summary>

```appsheet
"Loan (Statement-Based)",
[Principal_Balance] + [Interest_Balance],
```

</details>

---

# 🧾 Ledger Affects Balance?

<details open>
<summary><strong>Recommended Adjustment</strong></summary>

The key repair is allowing `Principal Applied` rows to affect the loan balance even when `Debt Charge ID` is blank.

```appsheet
OR(
  AND(
    ISNOTBLANK([Debt Charge ID]),
    IN(
      [Type],
      LIST(
        "Debt Payment",
        "Debt Payment Reversal",
        "Bill Payment",
        "Bill Payment Reversal",
        "Adjustment"
      )
    )
  ),

  AND(
    [Type] = "Principal Applied",
    [Allocation Type] = "Principal",
    ISNOTBLANK([Debt ID])
  )
)
```

</details>

---

# ⚠️ Notes

<details>
<summary><strong>Physical Formula Columns</strong></summary>

If any of these debt fields are physical columns with formulas, the Debt row may need to be refreshed or edited after formula or ledger repair work.

</details>
