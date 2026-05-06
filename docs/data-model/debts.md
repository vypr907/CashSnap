# 💳 Debts Table

---

## purpose

The **Debts** table represents all debt accounts tracked in CashSnap.

Examples:

* Credit cards
* Loans
* Installments
* Personal debts

---

## core-concept

```text
Ledger → Derived Balances → Debts
```

Debts do NOT directly calculate balances — they summarize Ledger.

---

## table-role

Debts act as the container for:

* Statements
* Payments
* PaymentAllocations
* Debt Charges
* Ledger activity

---

## column-inventory

<details>
<summary>View Column Inventory</summary>

| Column            | Type       | App Formula                                | Initial Value | Notes              |
| ----------------- | ---------- | ------------------------------------------ | ------------- | ------------------ |
| Row ID            | Text (Key) |                                            | `UNIQUEID()`  | Primary key        |
| Debt Name         | Text       |                                            |               | Display            |
| Debt Type         | Enum       |                                            |               | Loan / Credit Card |
| Model             | Enum       |                                            |               | Drives behavior    |
| Pay To            | Text / Ref |                                            |               | Payee              |
| Starting Balance  | Price      |                                            |               | Original balance   |
| Interest Rate     | Decimal    |                                            |               | Optional           |
| Minimum Payment   | Price      |                                            |               | Optional           |
| Principal Balance | Price (VC) | [View Formula](#principal-balance-formula) |               | Derived            |
| Interest Balance  | Price (VC) | [View Formula](#interest-balance-formula)  |               | Derived            |
| Remaining Balance | Price (VC) | [View Formula](#remaining-balance-formula) |               | Derived            |
| Paid So Far       | Price      |                                            |               | Needs clarity      |
| Status            | Enum       |                                            |               | Active / Paid      |

</details>

---

## key-virtual-columns

<details>
<summary>View Virtual Columns</summary>

| Column            | Type  | Formula                                    | Notes             |
| ----------------- | ----- | ------------------------------------------ | ----------------- |
| Principal Balance | Price | [View Formula](#principal-balance-formula) | Core balance      |
| Interest Balance  | Price | [View Formula](#interest-balance-formula)  | Interest tracking |
| Remaining Balance | Price | [View Formula](#remaining-balance-formula) | Total             |

</details>

---

## formulas

---

### principal-balance-formula

<details>
<summary>View Formula</summary>

```appsheet
[Starting Balance]
- SUM(
  SELECT(
    Ledger[Signed Amount],
    AND(
      [Debt ID] = [_THISROW].[Row ID],
      [Allocation Type] = "Principal",
      [Affects Balance?] = TRUE
    )
  )
)
```

</details>

---

### interest-balance-formula

<details>
<summary>View Formula</summary>

```appsheet
SUM(
  SELECT(
    Ledger[Signed Amount],
    AND(
      [Debt ID] = [_THISROW].[Row ID],
      [Charge Category] = "Interest",
      [Affects Balance?] = TRUE
    )
  )
)
```

</details>

---

### remaining-balance-formula

<details>
<summary>View Formula</summary>

```appsheet
[Principal Balance] + [Interest Balance]
```

</details>

---

## models

<details>
<summary>View Debt Models</summary>

### loan-statement-based

* Uses Statements
* Uses PaymentAllocations
* Split required

---

### loan-transaction-based

* Driven by transactions
* Still allocation-based

---

### credit-card

* Statement-driven
* Purchases may be Debt Charges

---

### installment

* Fixed schedule
* Optional charge modeling

---

### personal

* Simplified

</details>

---

## relationships

<details>
<summary>View Relationships</summary>

* Statements
* Payments
* PaymentAllocations
* Debt Charges
* Ledger

</details>

---

## rules

<details>
<summary>View Rules</summary>

### ledger-is-authoritative

Balances must match Ledger.

---

### principal-vs-interest

Must remain separate.

---

### payment-not-equal-principal

Avoid using full payment amount.

</details>

---

## known-issues

<details>
<summary>View Known Issues</summary>

| Issue            | Cause                |
| ---------------- | -------------------- |
| Balance mismatch | Ledger inconsistency |
| Double principal | Duplicate ledger     |
| Interest stuck   | Missing allocation   |

</details>

---

## debug-strategy

<details>
<summary>View Debug Strategy</summary>

1. Filter Ledger by Debt ID
2. Group by Allocation Type
3. Validate totals

</details>

---

## related-docs

* `docs/data-model/ledger.md`
* `docs/workflows/payment-allocation-flow.md`
