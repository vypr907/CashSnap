# 🧱 Master Tables Overview

---

## purpose

Defines all tables across the CashSnap ecosystem, grouped by app and role.

---

## system-overview

CashSnap ecosystem consists of:

* CashSnap → Financial engine (primary system)
* MedRecord → Medical + invoice system
* Armoury → Inventory / asset system
* ReceiptKeeper → Receipt capture system

---

## master-tables

<details>
<summary>View Tables</summary>

| App           | Table              | Description                       | Role          | Status         |
| ------------- | ------------------ | --------------------------------- | ------------- | -------------- |
| CashSnap      | Ledger             | Financial source of truth         | Core / SSOT   | Current        |
| CashSnap      | Payments           | Cash input events                 | Core          | Current        |
| CashSnap      | PaymentAllocations | Debt payment splits               | Core          | Current        |
| CashSnap      | Transaction Links  | Bill payment allocations          | Core          | Current        |
| CashSnap      | Debts              | Debt accounts                     | Core          | Current        |
| CashSnap      | Debt Charges       | Interest/fees                     | Core          | Current        |
| CashSnap      | Bills              | Recurring obligations             | Core          | Current        |
| CashSnap      | Bill Charges       | Bill instances                    | Core          | Current        |
| CashSnap      | Statements         | Debt billing periods              | Core          | Current        |
| CashSnap      | Debug Log          | System logging                    | Observability | Current        |
| MedRecord     | Appointments       | Medical visits                    | Core          | Needs Refactor |
| MedRecord     | Org_Invoices       | Medical invoices → CashSnap debts | Core          | Needs Refactor |
| MedRecord     | Medical_Expenses   | Expense tracking                  | Financial     | Needs Refactor |
| MedRecord     | Providers          | Provider directory                | Reference     | Needs Refactor |
| Armoury       | Entities           | Shared organization model         | Reference     | Needs Refactor |
| Armoury       | Entity_Roles       | Entity role system                | Reference     | Needs Refactor |
| Armoury       | Categories         | Inventory categorization          | Reference     | Needs Review   |
| ReceiptKeeper | Receipts           | Receipt capture                   | Core          | Needs Review   |
| ReceiptKeeper | Tags               | Receipt categorization            | Reference     | Needs Review   |

</details>

---

## notes

* CashSnap is the most optimized system
* Other apps require schema alignment
* Entity system is incomplete and needs redesign

---

## related-docs

* `docs/app-ecosystem/cross-app-flows.md`
* `docs/app-ecosystem/entity-system.md`
