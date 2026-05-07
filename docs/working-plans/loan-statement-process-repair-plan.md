# Loan Statement Process Repair Plan

<a id="loan-statement-process-repair-plan"></a>

## Purpose

<a id="purpose"></a>

This plan repairs CashSnap's statement-based loan process so:

- `Statements` and `Loan_Pmt_Splits` define the statement/payment split instructions.
- `PaymentAllocations` store the applied Principal, Interest, and Fee split records.
- `Debt Charges` represent Interest/Fee amounts owed.
- `Ledger` remains the financial source of truth.

---

<details open>
<summary><strong>Current Progress</strong></summary>

<a id="current-progress"></a>

## Completed

<a id="completed"></a>

- Step 1.1 / Step 3.4: Disabled the `Split Happens` bot because it only created duplicate Interest charges from `Loan_Pmt_Splits`.
- Step 7.4: Created all missing `Principal Applied` ledger rows for the current repair pass.

## Next Recommended Step

<a id="next-recommended-step"></a>

Continue with Step 2.1: update the general `Statement Bot` so it does not run for debts where:

```appsheet
[Debt_ID].[Model] = "Loan (Statement-Based)"
```

Recommended general Statement Bot condition:

```appsheet
OR(
  ISBLANK([Debt_ID].[Model]),
  [Debt_ID].[Model] <> "Loan (Statement-Based)"
)
```

</details>

---

<details>
<summary><strong>Phase 1 — Freeze and Inventory Current Behavior</strong></summary>

<a id="phase-1-freeze-and-inventory-current-behavior"></a>

## Step 1.1 — Disable Duplicate Loan Automation Paths

<a id="step-1-1-disable-duplicate-loan-automation-paths"></a>

Temporarily disable or condition-block duplicate loan automation paths before testing.

Primary target:

```text
Loan_Pmt_Splits → Add Interest Charge to Splits
```

Status: **Completed** — `Split Happens` bot disabled.

## Step 1.2 — Pick One Test Statement/Payment Set

<a id="step-1-2-pick-one-test-statement-payment-set"></a>

Document the expected output for one clean statement/payment test:

- 1 `Statement`
- Optional 1 `Context`
- 1 `Loan_Pmt_Split` per payment/split
- 2 `PaymentAllocations` per payment:
  - Interest
  - Principal
- 1 Interest `Debt Charge`
- Normally 3 `Ledger` rows:
  - `+Interest Debt Charge`
  - `-Interest Payment`
  - `-Principal Applied`

## Step 1.3 — Identify Old Full-Payment Debt Ledgers

<a id="step-1-3-identify-old-full-payment-debt-ledgers"></a>

Find old full-payment `Debt Payment` ledger rows and decide whether to:

- Delete them during repair, or
- Set `Affects Balance? = FALSE` so they do not double-count against principal.

</details>

---

<details>
<summary><strong>Phase 2 — Clean Up Statement Bot Ownership</strong></summary>

<a id="phase-2-clean-up-statement-bot-ownership"></a>

## Step 2.1 — Exclude Loan Statements from General Statement Bot

<a id="step-2-1-exclude-loan-statements-from-general-statement-bot"></a>

The general `Statement Bot` should not process `Loan (Statement-Based)` debts unless intentionally shared.

Recommended condition:

```appsheet
OR(
  ISBLANK([Debt_ID].[Model]),
  [Debt_ID].[Model] <> "Loan (Statement-Based)"
)
```

## Step 2.2 — Prevent Payment Repair Bot Duplication

<a id="step-2-2-prevent-payment-repair-bot-duplication"></a>

`Payment Repair Bot` should not duplicate `Loan Statement Bot` payment attachment work.

Preferred design:

```text
Payment Repair Bot = Credit Card or orphan repair only
Loan Statement Bot = statement-based loan processing owner
```

## Step 2.3 — Make Loan Statement Bot the Loan Orchestrator

<a id="step-2-3-make-loan-statement-bot-the-loan-orchestrator"></a>

`Loan Statement Bot` should own:

- Attaching payments
- Creating context
- Running allocation process
- Locking context
- Logging start/finish

</details>

---

<details>
<summary><strong>Phase 3 — Redefine Loan_Pmt_Splits Role</strong></summary>

<a id="phase-3-redefine-loan-pmt-splits-role"></a>

## Step 3.1 — Keep Loan_Pmt_Splits as Split Instructions

<a id="step-3-1-keep-loan-pmt-splits-as-split-instructions"></a>

Keep `Loan_Pmt_Splits` as the split instruction/source table, especially when split details are entered directly from the Add Loan Statement form.

## Step 3.2 — Recommended Loan_Pmt_Splits Columns

<a id="step-3-2-recommended-loan-pmt-splits-columns"></a>

Recommended columns:

- Debt
- Statement
- Payment
- Payment Date
- Total Payment Amount
- Interest
- Principal
- Processed?
- ContextID / ProcessRunID, if available
- Links to created `PaymentAllocations`, if useful

## Step 3.3 — Stop Direct Interest Charge Creation

<a id="step-3-3-stop-direct-interest-charge-creation"></a>

`Loan_Pmt_Splits` should not directly create Interest `Debt Charges` if the `PaymentAllocation` bot creates or links those charges.

## Step 3.4 — Disable Split Happens for Normal Processing

<a id="step-3-4-disable-split-happens-for-normal-processing"></a>

Disable or condition-block:

```text
Split Happens → Add Interest Charge to Splits
```

Status: **Completed** — bot disabled.

</details>

---

<details>
<summary><strong>Phase 4 — PaymentAllocation Process Rules</strong></summary>

<a id="phase-4-paymentallocation-process-rules"></a>

## Step 4.1 — PaymentAllocations Are Applied Split Records

<a id="step-4-1-paymentallocations-are-applied-split-records"></a>

`PaymentAllocations` should be created from the `Loan_Pmt_Splits` / Statement workflow.

## Step 4.2 — Interest Allocation Behavior

<a id="step-4-2-interest-allocation-behavior"></a>

For Interest allocations:

1. Find or create exactly one Interest `Debt Charge` for the statement/payment interest amount.
2. Set `PaymentAllocations[Debt Charge ID]`.
3. Create an Interest Payment ledger linked to:
   - PaymentAllocation
   - Payment
   - Debt
   - Statement
   - Debt Charge

## Step 4.3 — Fee Allocation Behavior

<a id="step-4-3-fee-allocation-behavior"></a>

Same as Interest, but with:

```text
ChargeType / AllocationType = Fee
```

## Step 4.4 — Principal Allocation Behavior

<a id="step-4-4-principal-allocation-behavior"></a>

For Principal allocations:

- Do not create a `Debt Charge`.
- Create a `Principal Applied` ledger linked to:
  - PaymentAllocation
  - Payment
  - Debt
  - Statement
- Keep `Debt Charge ID` blank.

Status: **Completed for missing historical principal ledgers in current repair pass.**

## Step 4.5 — Do Not Put Principal in Charge Category

<a id="step-4-5-do-not-put-principal-in-charge-category"></a>

Principal should not populate `Charge Category` unless the enum is intentionally expanded.

Preferred values:

```text
Allocation Type = Principal
Charge Category = blank
```

</details>

---

<details>
<summary><strong>Phase 5 — Ledger Model</strong></summary>

<a id="phase-5-ledger-model"></a>

## Step 5.1 — Debt Charge Bot Creates Charge Ledgers

<a id="step-5-1-debt-charge-bot-creates-charge-ledgers"></a>

`New Debt Charge Bot` creates the `+Interest` / `+Fee` ledger when a `Debt Charge` is added.

## Step 5.2 — Pmt Allocation Bot Creates Payment-Side Ledgers

<a id="step-5-2-pmt-allocation-bot-creates-payment-side-ledgers"></a>

`Pmt Allocation Bot` creates:

- `-Interest Payment`
- `-Principal Applied`

## Step 5.3 — Required Ledger Links

<a id="step-5-3-required-ledger-links"></a>

Ledger rows should include:

- Debt ID
- PaymentID, if applicable
- StatementID, if applicable
- PmtAllocID, if applicable
- Debt Charge ID for Interest/Fee only
- Allocation Type
- Charge Category for Interest/Fee only
- Amount / Signed Amount
- Affects Balance?
- Src Row ID / TraceID

## Step 5.4 — Prevent Full-Payment Double Counting

<a id="step-5-4-prevent-full-payment-double-counting"></a>

Full-payment debt ledgers should not also reduce the loan balance if split ledgers exist.

Options:

- Replace them with split ledgers, or
- Mark them `Affects Balance? = FALSE`.

</details>

---

<details>
<summary><strong>Phase 6 — Formula Alignment for Loan (Statement-Based)</strong></summary>

<a id="phase-6-formula-alignment-for-loan-statement-based"></a>

## Step 6.1 — Principal_Balance

<a id="step-6-1-principal-balance"></a>

`Principal_Balance` should equal:

```text
Starting Balance - Principal allocations / Principal Applied ledgers + relevant adjustments
```

## Step 6.2 — Interest Charged

<a id="step-6-2-interest-charged"></a>

`Interest Charged` should sum Interest `Debt Charges`, not principal allocations.

## Step 6.3 — Interest_Balance

<a id="step-6-3-interest-balance"></a>

`Interest_Balance` should equal:

```text
Interest Charges - Interest Payment allocations / ledgers
```

## Step 6.4 — Remaining Balance

<a id="step-6-4-remaining-balance"></a>

For statement-based loans:

```text
Remaining Balance = Principal_Balance + Interest_Balance
```

## Step 6.5 — Paid So Far Meaning

<a id="step-6-5-paid-so-far-meaning"></a>

Clarify whether `Paid So Far` means:

- Total cash paid, or
- Principal paid

Avoid using total payment amount as principal reduction.

</details>

---

<details>
<summary><strong>Phase 7 — Repair Existing Auto Loan Data</strong></summary>

<a id="phase-7-repair-existing-auto-loan-data"></a>

## Step 7.1 — Verify PaymentAllocations Exist

<a id="step-7-1-verify-paymentallocations-exist"></a>

For each existing `Loan_Pmt_Split`, verify corresponding Interest and Principal `PaymentAllocations` exist.

## Step 7.2 — Remove or Neutralize Duplicate Interest Charges

<a id="step-7-2-remove-or-neutralize-duplicate-interest-charges"></a>

Remove or neutralize duplicate Interest `Debt Charges` created by both:

- `Loan_Pmt_Splits`
- `PaymentAllocations`

## Step 7.3 — Link Interest PaymentAllocations to Surviving Debt Charge

<a id="step-7-3-link-interest-paymentallocations-to-surviving-debt-charge"></a>

Each Interest `PaymentAllocation` should point to the correct surviving Interest `Debt Charge`.

## Step 7.4 — Create Missing Principal Applied Ledgers

<a id="step-7-4-create-missing-principal-applied-ledgers"></a>

Create/link missing `Principal Applied` ledger rows for Principal allocations.

Status: **Completed for current repair pass.**

## Step 7.5 — Create Missing Interest Payment Ledgers

<a id="step-7-5-create-missing-interest-payment-ledgers"></a>

Create/link missing Interest Payment ledger rows for Interest allocations.

## Step 7.6 — Neutralize Old Full-Payment Debt Payment Ledgers

<a id="step-7-6-neutralize-old-full-payment-debt-payment-ledgers"></a>

Mark old full-payment `Debt Payment` ledger rows:

```text
Affects Balance? = FALSE
```

Or remove them if safe.

## Step 7.7 — Run Ledger Repair Action

<a id="step-7-7-run-ledger-repair-action"></a>

Run the Ledger repair action to populate missing:

- Charge Category
- PmtAllocID
- Allocation Type
- StatementID

</details>

---

<details>
<summary><strong>Phase 8 — Testing and Rollout</strong></summary>

<a id="phase-8-testing-and-rollout"></a>

## Step 8.1 — Test One Statement and One Payment First

<a id="step-8-1-test-one-statement-and-one-payment-first"></a>

Use one controlled statement/payment set before re-enabling broader automation.

## Step 8.2 — Expected Per Payment

<a id="step-8-2-expected-per-payment"></a>

Expected rows per payment:

- 1 Principal allocation
- 1 Interest allocation
- 1 Interest `Debt Charge`
- 1 `+Interest Charge` ledger
- 1 `-Interest Payment` ledger
- 1 `-Principal Applied` ledger

## Step 8.3 — Confirm No Duplicate Interest Debt Charges

<a id="step-8-3-confirm-no-duplicate-interest-debt-charges"></a>

There should be exactly one Interest `Debt Charge` for the intended statement/payment interest amount.

## Step 8.4 — Confirm Principal_Balance Decreases Correctly

<a id="step-8-4-confirm-principal-balance-decreases-correctly"></a>

`Principal_Balance` should decrease only by principal amount.

## Step 8.5 — Confirm Interest_Balance Clears When Paid

<a id="step-8-5-confirm-interest-balance-clears-when-paid"></a>

`Interest_Balance` should return to zero when interest is paid.

## Step 8.6 — Re-enable Automation Carefully

<a id="step-8-6-re-enable-automation-carefully"></a>

Re-enable automation only after the one-payment test passes.

## Step 8.7 — Add Debug Logs

<a id="step-8-7-add-debug-logs"></a>

Add debug logs using the CashSnap Debug Log standard around:

- Statement Bot start/finish
- Payment attachment
- Allocation creation
- Charge creation/linking
- Ledger creation
- Context lock

</details>
