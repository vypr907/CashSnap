# 💵 Payments Table

---

## purpose

The **Payments** table represents cash entering the system.

---

## core-concept

```text
Payment = cash event
Allocation = accounting effect
Ledger = financial truth
```

---

## table-role

Payments:

* Trigger allocation flows
* Link to Bills or Debts
* Drive Ledger creation

---

## column-inventory

<details>
<summary>View Column Inventory</summary>

| Column                | Type   | App Formula                                    | Initial Value | Notes           |
| --------------------- | ------ | ---------------------------------------------- | ------------- | --------------- |
| Row ID                | Text   |                                                | `UNIQUEID()`  | Key             |
| Payment Name          | Text   |                                                |               | Display         |
| Amount Paid           | Price  |                                                |               | Amount          |
| Date                  | Date   |                                                |               | Payment date    |
| Type                  | Enum   |                                                |               | Bill / Debt     |
| Bill ID               | Ref    |                                                |               | Optional        |
| Debt ID               | Ref    |                                                |               | Optional        |
| Cleared               | Yes/No |                                                |               | Status          |
| Cleared Date          | Date   |                                                |               |                 |
| Payment Status        | Enum   |                                                |               | Pending/Cleared |
| Statement_ID          | Ref    |                                                |               | Optional        |
| Reverses Payment      | Ref    |                                                |               |                 |
| Return Date           | Date   |                                                |               |                 |
| Remaining To Allocate | Price  | [View Formula](#remaining-to-allocate-formula) |               | Core            |
| Processed             | Yes/No |                                                |               | Bot gating      |

</details>

---

## key-virtual-columns

<details>
<summary>View Virtual Columns</summary>

| Column                | Type  | Formula                                        |
| --------------------- | ----- | ---------------------------------------------- |
| Remaining To Allocate | Price | [View Formula](#remaining-to-allocate-formula) |

</details>

---

## formulas

---

### remaining-to-allocate-formula

<details>
<summary>View Formula</summary>

```appsheet
[Amount Paid]
- SUM(
  SELECT(
    Transaction Links[Amount Applied],
    [Payment ID] = [_THISROW].[Row ID]
  )
)
- SUM(
  SELECT(
    PaymentAllocations[Amount],
    [PaymentID] = [_THISROW].[Row ID]
  )
)
```

</details>

---

## flows

<details>
<summary>View Payment Flow</summary>

Payment Created
→ Allocation begins
→ Links / Allocations created
→ Ledger rows created
→ Fully allocated

</details>

---

## rules

<details>
<summary>View Rules</summary>

### must-be-allocated

Remaining To Allocate must reach 0

---

### cleared-matters

Only cleared payments should count

---

### no-direct-balance-impact

Must go through allocations

</details>

---

## known-issues

<details>
<summary>View Known Issues</summary>

| Issue       | Cause             |
| ----------- | ----------------- |
| Not applied | Allocation failed |
| Partial     | Remaining stuck   |
| Duplicate   | Bot re-trigger    |

</details>

---

## debug-strategy

<details>
<summary>View Debug Strategy</summary>

Check:

* Transaction Links
* PaymentAllocations
* Ledger

</details>

---

## related-docs

* `docs/data-model/ledger.md`
* `docs/workflows/new-payment-flow.md`
