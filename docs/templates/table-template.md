# 📋 {TABLE_NAME}.md

> Last Updated: 2026-05-08
>
> Related Docs:
>
> - [related-process.md](../processes/related-process.md)
> - [related-bot.md](../bots/related-bot.md)

---

# 📌 Overview

## Purpose

Describe the purpose of the table.

Example:

- Stores financial transactions
- Stores payment allocations
- Stores statement periods
- Stores ledger entries

---

# 🧠 Design Philosophy

<details open>
<summary><strong>Expand Design Notes</strong></summary>

Describe:

- Why this table exists
- What system owns this data
- What SHOULD be stored here
- What SHOULD NOT be stored here
- Relationship to Ledger / Statements / Payments / Charges

</details>

---

# 🗂️ Table Relationships

<details open>
<summary><strong>Expand Relationship Diagram</strong></summary>

```text
Payments
  ↓
PaymentAllocations
  ↓
Ledger
```

</details>

---

# 🔑 Core Columns

<details open>
<summary><strong>Expand Core Columns</strong></summary>

| Column | Type | Purpose |
|---|---|---|
| Row ID | Text | Primary key |
| Created At | DateTime | Audit timestamp |
| Status | Enum | Processing state |

</details>

---

# 📚 ALL Columns

<details open>
<summary><strong>Expand Full Column Reference</strong></summary>

## Example Column

### Column Name

```text
Example_Column
```

### Type

```text
Text
```

### Purpose

Describe the purpose of the column.

### Formula

```appsheet
{FORMULA}
```

### Notes

- Important behavior
- Edge cases
- Performance concerns

---

## Repeat Section For Every Column

</details>

---

# ⚙️ Virtual Columns

<details>
<summary><strong>Expand Virtual Columns</strong></summary>

Document all virtual columns separately.

</details>

---

# 🧮 Formula Reference

<details>
<summary><strong>Expand Formula Reference</strong></summary>

## Example Formula

```appsheet
SUM(
  SELECT(
    Ledger[Signed Amount],
    [Debt ID] = [_THISROW].[Row ID]
  )
)
```

</details>

---

# 🔄 Automation Dependencies

<details open>
<summary><strong>Expand Automation Dependencies</strong></summary>

## Bots

- Statement Bot
- Loan Statement Bot
- LedgerBot

---

## Actions

- Create Ledger
- Repair Missing Fields

---

## Processes

- Statement Reconciliation
- Payment Allocation

</details>

---

# 🧪 Testing Checklist

<details open>
<summary><strong>Expand Testing Checklist</strong></summary>

- [ ] Formula works correctly
- [ ] Related rows populate
- [ ] No circular references
- [ ] Ledger values reconcile
- [ ] Statement values reconcile
- [ ] Reversals behave correctly

</details>

---

# 🐞 Known Issues / Edge Cases

<details>
<summary><strong>Expand Known Issues</strong></summary>

- Duplicate records
- Missing references
- Circular dependency risk
- Performance-heavy SELECTs

</details>

---

# 🚀 Future Improvements

<details>
<summary><strong>Expand Future Improvements</strong></summary>

- [ ] Improve performance
- [ ] Reduce virtual column load
- [ ] Add integrity validators
- [ ] Improve repair tooling

</details>

---

# 🔍 Cross References

## Related Tables

- Payments
- Ledger
- Debt Charges

---

## Related Systems

- Ledger System
- Statement Reconciliation
- Payment Allocation System

---

# 📚 Change History

<details open>
<summary><strong>Expand Change History</strong></summary>

| Date | Change |
|---|---|
| 2026-05-08 | Initial table documentation template created |

</details>
