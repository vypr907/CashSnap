# CHANGELOG

<a id="changelog"></a>

All notable CashSnap changes should be documented here.

---

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
