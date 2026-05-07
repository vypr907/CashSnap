# CashSnap AI Context

---

## purpose

This document gives ChatGPT / AI assistants the current working context for the CashSnap ecosystem.

It should be updated whenever schema, workflows, app relationships, or major design decisions change.

---

## ecosystem-overview

CashSnap is part of a four-app AppSheet ecosystem:

| App           | Purpose                                                                            | Current State                     |
| ------------- | ---------------------------------------------------------------------------------- | --------------------------------- |
| CashSnap      | Financial engine for bills, debts, payments, ledger, statements, and allocations   | Most optimized                    |
| MedRecord     | Medical records, appointments, invoices, providers, medications, HSA, and expenses | Functional but needs refactor     |
| Armoury       | Inventory / asset management, item tracking, and entity modeling                   | Functional but needs optimization |
| ReceiptKeeper | Quick receipt capture and receipt tagging                                          | Lightweight / needs review        |

---

## architecture-principle

CashSnap is the primary financial source of truth.

Other apps may create or feed financial records into CashSnap, but CashSnap should own final financial state through:

```text
Payments → Allocations → Ledger → Balances
```

---

## cross-app-relationships

<details>
<summary>View Cross-App Relationships</summary>

### medrecord-to-cashsnap

MedRecord can create matching CashSnap debts from medical invoices or appointment-related invoices.

Target ownership:

```text
MedRecord invoice data
→ CashSnap Debt
→ CashSnap Payment / Allocation / Ledger
```

CashSnap owns financial tracking after the debt is created.

---

### receiptkeeper-to-armoury

ReceiptKeeper is intended for fast receipt capture.

Armoury is intended for inventory / asset tracking.

Target flow:

```text
ReceiptKeeper Receipt
→ Armoury Item records
→ Items linked back to Receipt
```

ReceiptKeeper captures the transaction artifact; Armoury owns the item/asset details.

---

### entity-system

The Entity tables were an early attempt to normalize organizations across apps.

Example:

```text
Safeway
→ Grocery Store
→ Pharmacy
→ Vendor
```

Target design:

```text
One Entity
→ Multiple Roles
→ Used across apps
```

</details>

---

## current-design-philosophy

<details>
<summary>View Design Philosophy</summary>

### ledger-as-source-of-truth

CashSnap is moving toward Ledger as the source of truth for:

* Bills
* Debts
* Charges
* Payments
* Adjustments
* Balances

---

### allocation-based-accounting

Payments should not directly change balances.

Instead:

```text
Payment
→ Transaction Links or PaymentAllocations
→ Ledger
→ Balance
```

---

### deterministic-automation

Bots should be:

* gated
* idempotent where possible
* traceable through Debug Log
* protected from duplicate execution

---

### documentation-first-maintenance

GitHub documentation should remain aligned with the app.

All `.md` docs should use:

* collapsible `<details>` sections
* internal anchor links
* cross-document links
* stable formula/header naming

</details>

---

## core-cashsnap-tables

<details>
<summary>View CashSnap Tables</summary>

| Table                | Purpose                                                                       |
| -------------------- | ----------------------------------------------------------------------------- |
| Accounts             | Payment methods, funding sources, and balance containers                      |
| Adjustments          | Manual financial corrections                                                  |
| Bills                | Recurring and statement-based obligations                                     |
| Bill Charges         | Individual bill obligations per cycle/period                                  |
| Payments             | Cash payment records                                                          |
| Transaction Links    | Bill-side payment allocations                                                 |
| Debts                | Debt accounts including loans, credit cards, installments, and personal debts |
| Debt Charges         | Interest, fees, penalties, and other non-principal debt charges               |
| Statements           | Statement periods for credit cards and statement-based loans                  |
| PaymentAllocations   | Debt-side payment splits                                                      |
| Loan_Pmt_Splits      | Loan payment split instruction rows                                           |
| Ledger               | Financial source of truth                                                     |
| Debug Log            | Structured automation logging                                                 |
| Context              | Workflow context for multi-step automation                                    |
| Pay Period           | Pay period planning windows                                                   |
| Paycheck Selector    | Active pay-period selector                                                    |
| Installment Schedule | Expected installment payment schedule                                         |
| Income               | Expected income sources                                                       |
| Income Deposits      | Actual income deposit records                                                 |
| Summary Totals       | Dashboard/reporting helper                                                    |
| UserSettings         | App configuration and preferences                                             |

</details>

---

## core-workflows

<details>
<summary>View Core Workflows</summary>

### bill-payment-flow

```text
Payment
→ Transaction Link
→ Ledger
→ Bill Charge balance
```

---

### debt-payment-flow

```text
Payment
→ PaymentAllocation
→ Debt Charge if Interest/Fee
→ Ledger
→ Debt balance
```

---

### loan-statement-flow

```text
Statement
→ Attach Payments
→ Loan_Pmt_Splits
→ PaymentAllocations
→ Debt Charges
→ Ledger
```

---

### medrecord-invoice-flow

```text
Appointment / Invoice
→ Org_Invoice
→ CashSnap Debt
→ CashSnap payment tracking
```

---

### receipt-inventory-flow

```text
ReceiptKeeper Receipt
→ Armoury Items
→ Inventory / asset records
```

</details>

---

## loan-statement-process-repair-plan

<details>
<summary>View Current Repair Plan</summary>

Current working plan:

1. Freeze duplicate loan automation paths.
2. Use Loan_Pmt_Splits as the split instruction/source table.
3. Use PaymentAllocations as the applied split records.
4. Create Debt Charges only for Interest/Fee.
5. Principal allocations create Principal Applied ledger rows only.
6. Debt Charge bot creates positive Interest/Fee ledger rows.
7. Pmt Allocation bot creates negative Interest/Fee payment and Principal Applied ledger rows.
8. Old full-payment Debt Payment ledger rows should not affect balance if split ledgers exist.
9. Test one statement/payment first before re-enabling automation.
10. Add Debug Logs around every major step.

Expected per loan payment:

| Record Type          | Expected Count                                             |
| -------------------- | ---------------------------------------------------------- |
| PaymentAllocations   | 2: Interest + Principal                                    |
| Interest Debt Charge | 1                                                          |
| Ledger Rows          | 3: +Interest Charge, -Interest Payment, -Principal Applied |

</details>

---

## debug-log-standard

<details>
<summary>View Debug Log Standard</summary>

Use current CashSnap Debug Log schema:

| Column         | Purpose                          |
| -------------- | -------------------------------- |
| Timestamp      | Log timestamp                    |
| Process        | Bot/system name                  |
| Action         | Action name when applicable      |
| Stage          | Logical step                     |
| Status         | START / SUCCESS / INFO / FAIL    |
| Source Table   | Origin table                     |
| Message        | Human-readable summary           |
| Details        | Structured key=value diagnostics |
| TraceID        | Links related logs               |
| Row Identifier | Source row key                   |
| Bill ID        | Related bill                     |
| Debt ID        | Related debt                     |
| TransLink ID   | Related transaction link         |
| Ledger ID      | Related ledger row               |
| ProcessRunID   | Groups one execution             |

Message format:

```text
[Process] | [Stage] | [Status] | [SourceTable]:[Row Identifier] | [Summary]
```

Details format:

```text
BillID=... | DebtID=... | PaymentID=... | Amt=... | TraceID=...
```

</details>

---

## documentation-standard

<details>
<summary>View Documentation Standard</summary>

All markdown files should use:

### required-format

* Lowercase, hyphenated section headers
* Collapsible sections using `<details>`
* Internal anchor links for formulas and referenced sections
* Cross-document links for related docs
* Column inventory sections for table docs
* Formula sections with stable names

### formula-section-pattern

```md
| Signed Amount | Price (VC) | [View Formula](#signed-amount-formula) |  | Notes |
```

````md
### signed-amount-formula

<details>
<summary>View Formula</summary>

```appsheet
...
````

</details>
```

### recommended-sections-for-table-docs

```text
purpose
core-concept
table-role
column-inventory
key-virtual-columns
formulas
actions
bots
relationships
rules
known-issues
debug-strategy
repair-notes
related-docs
```

</details>

---

## known-issues

<details>
<summary>View Known Issues</summary>

| Area          | Issue                                                                        |
| ------------- | ---------------------------------------------------------------------------- |
| Bots          | Some flows may double-fire                                                   |
| Ledger        | Legacy full-payment debt ledger rows can double-count principal              |
| Loans         | Interest charges can duplicate if both splits and allocations create charges |
| Statements    | Payments/reversals may belong to different statement periods                 |
| Formulas      | Circular dependencies can occur in bill cycle logic                          |
| Performance   | Debug Log rows and repeated SELECT() formulas can slow sync                  |
| Cross-App     | MedRecord, Armoury, and ReceiptKeeper are less optimized than CashSnap       |
| Entity System | Organization/entity model needs redesign                                     |

</details>

---

## current-priorities

<details>
<summary>View Current Priorities</summary>

1. Stabilize CashSnap Ledger SSOT.
2. Repair Loan Statement workflow.
3. Stabilize PaymentAllocation workflow.
4. Finish table documentation with column inventories and formulas.
5. Document cross-app flows.
6. Redesign Entity system.
7. Refactor MedRecord financial tables to align with CashSnap.
8. Clarify ReceiptKeeper → Armoury item flow.
9. Reduce Debug Log and SELECT() performance overhead.

</details>

---

# AI Context — Loan Statement Process Update

<a id="ai-context-loan-statement-process-update"></a>

<details open>
<summary><strong>Context Summary</strong></summary>

<a id="context-summary"></a>

CashSnap is repairing the `Loan (Statement-Based)` workflow for the auto loan. The intended model is:

```text
Statements + Loan_Pmt_Splits = split instructions
PaymentAllocations = applied split records
Debt Charges = interest/fee owed
Ledger = financial source of truth
```

The repair plan is documented in:

- [Loan Statement Process Repair Plan](docs/processes/loan-statement-process-repair-plan.md#loan-statement-process-repair-plan)

</details>

---

<details>
<summary><strong>Current Repair Progress</strong></summary>

<a id="current-repair-progress"></a>

## Completed

<a id="completed"></a>

- Disabled `Split Happens` because it only created duplicate Interest `Debt Charges` from `Loan_Pmt_Splits`.
- Created all missing `Principal Applied` ledgers for current Principal `PaymentAllocations`.

## In Progress / Next

<a id="in-progress-next"></a>

- Update the general `Statement Bot` condition so it does not run on `Loan (Statement-Based)` debts.
- Continue repairing missing Interest Payment ledgers and allocation/charge links.
- Neutralize old full-payment `Debt Payment` ledgers if split ledgers exist.

</details>

---

<details>
<summary><strong>Important Formulas / Conditions</strong></summary>

<a id="important-formulas-conditions"></a>

## General Statement Bot Exclusion

<a id="general-statement-bot-exclusion"></a>

Use this condition to prevent the general `Statement Bot` from processing loan statements:

```appsheet
OR(
  ISBLANK([Debt_ID].[Model]),
  [Debt_ID].[Model] <> "Loan (Statement-Based)"
)
```

## Loan Statement Bot Condition

<a id="loan-statement-bot-condition"></a>

The dedicated `Loan Statement Bot` condition is:

```appsheet
[Debt_ID].[Model] = "Loan (Statement-Based)"
```

</details>


---

## related-docs

* `README.md`
* `docs/app-ecosystem/master-tables.md`
* `docs/app-ecosystem/cross-app-flows.md`
* `docs/app-ecosystem/entity-system.md`
* `docs/app-ecosystem/optimization-roadmap.md`
* `docs/data-model/ledger.md`
* `docs/data-model/debts.md`
* `docs/data-model/payments.md`
* `docs/data-model/bills.md`
* `docs/workflows/loan-statement-flow.md`
* `docs/workflows/payment-allocation-flow.md`
