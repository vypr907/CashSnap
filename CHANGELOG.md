# 📚 CHANGELOG

<a id="changelog"></a>

> All notable CashSnap changes should be documented here.
>
> This changelog is intentionally append-friendly.
>
> Add new entries to the TOP of the appropriate section rather than regenerating the entire document.


---
# 🚀 Recent Changes

<details open>
<summary><strong>2026-05-08 — Statement-Based Loan Ledger Repair Progress</strong></summary>

<a id="2026-05-08-statement-based-loan-ledger-repair-progress"></a>

### Summary

Continued repair work on CashSnap statement-based loan processing, focused on the Auto Loan debt. The system is being aligned toward the latest Ledger-as-source-of-truth model where PaymentAllocations remain process/audit records, Debt Charges represent interest/fee obligations, and Ledger drives calculated balances.

### Changed

- Updated loan balance logic direction so `Principal_Balance`, `Interest_Balance`, `Interest Charged`, `Interest Paid`, and `Principal Paid` should read primarily from Ledger-derived values.
- Confirmed `Principal Applied` ledger rows are the correct mechanism for reducing principal on `Loan (Statement-Based)` debts.
- Adjusted `Affects Balance?` logic so `Principal Applied` rows can affect loan balances even when `Debt Charge ID` is blank.
- Confirmed that `Loan (Statement-Based)` principal should not require a principal Debt Charge.
- Clarified that statement-based loan interest should flow through `Loan_Pmt_Splits` → `PaymentAllocations` → `Debt Charges` → `Ledger`.

### Fixed

- Identified stale physical formula columns as a source of confusing loan balance results; refreshing the Debt row allowed formulas to recalculate correctly.
- Resolved principal-side calculation mismatch:
  - Starting Balance: `20524.00`
  - Principal Paid: `10272.16`
  - Principal Balance: `10251.84`
  - Ledger Balance: `10251.84`
- Confirmed remaining issue is interest-side repair:
  - Interest Charged: `2845.41`
  - Interest Paid: `122.45`
  - Interest Balance: `2722.96`
- Determined `Interest Paid` is likely only detecting a subset of interest payment ledger rows due to missing or inconsistent fields such as `Allocation Type`, `Charge Category`, `Debt Charge ID`, or Ledger `Type`.

### Related Docs

- `docs/processes/loan-statement-process-repair-plan.md`
- `docs/processes/auto-loan-repair-status.md`
- `docs/formulas/debt-ledger-ssot-formulas.md`
- `docs/tables/ledger.md`
- `docs/tables/debts.md`
- `docs/tables/statements.md`
- `docs/bots/pmt-allocation-bot.md`

</details>

<details open>
<summary><strong>2026-05-08 — Documentation Overhaul</strong></summary>

<a id="2026-05-08-documentation-overhaul"></a>

### Documentation

- Added standardized table documentation template
- Added `statements.md` using new table template
- Added `/docs/templates/` folder for reusable documentation templates
- Standardized CashSnap bot/process/table documentation styling
- Added toggle-heavy markdown structure with Mermaid placeholders and testing checklists

---

# 📂 Documentation Structure

```text
/docs
  /bots
  /processes
  /tables
  /templates
```

---

# 🧠 Documentation Philosophy

- Append new changes instead of rewriting history
- Keep newest entries at the top
- Prefer small focused updates
- Use collapsible sections for readability
- Keep formulas and Mermaid diagrams embedded with related systems

</details>


<details open>
<summary><strong>2026-05-07 — Loan Statement Process Repair</strong></summary>

<a id="2026-05-07-loan-statement-process-repair"></a>

## Summary

Repaired the statement-based loan workflow so loan statements, split rows, payment allocations, debt charges, and ledger rows work together without duplicate interest charges or missing principal ledger activity.

## Changed

- Disabled the `Split Happens` bot because its only active behavior was creating duplicate interest `Debt Charges` from `Loan_Pmt_Splits`.
- Updated the intended role of `Loan_Pmt_Splits` to act as split instructions only.
- Confirmed `PaymentAllocations` should own applied allocation records for Interest and Principal.
- Added missing `Principal Applied` ledger rows for existing Principal allocations.
- Added a condition to prevent the general `Statement Bot` from running on `Loan (Statement-Based)` debts.

## Fixed

- Prevented duplicate Interest `Debt Charges` from being created by both `Loan_Pmt_Splits` and `PaymentAllocations`.
- Restored missing principal ledger activity for statement-based loan payments.
- Reduced risk of double-counting loan payments against principal.

## Related Docs

- [Loan Statement Process Repair Plan](docs/processes/loan-statement-process-repair-plan.md#loan-statement-process-repair-plan)
- [AI Context Update](AI_CONTEXT.loan-statement-process-update.md#ai-context-loan-statement-process-update)

</details>
