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

| App / System | Table | Description | Source | Role | Optimization Status |
|---|---|---|---|---|---|
| CashSnap | Accounts | Financial accounts used as payment methods, funding sources, or account/balance containers | CashSnap | Reference / Financial | Needs Review |
| CashSnap | [Adjustments](adjustments.md) | Manual financial corrections applied to bills, debts, statements, or ledger reconciliation | CashSnap | Financial / Repair | Current-ish |
| MedRecord | Appointments | Medical appointments and visits | MedData | Core / Medical | Legacy-ish |
| MedRecord | ApptIssues | Issues, notes, concerns, or follow-up items tied to appointments | MedData | Support / Medical | Legacy-ish |
| CashSnap | [Bill Charges](bill-charges.md) | Individual bill obligations generated per billing cycle or statement period | CashSnap | Core / Financial | Current |
| CashSnap | [Bills](bills.md) | Recurring and statement-based financial obligations | CashSnap | Core / Financial | Current |
| CashSnap | Categories | Financial categories for bills, debts, income, and reporting | CashSnap | Reference / Financial | Needs Review |
| Armoury | Categories | Inventory or asset categories used by Armoury | Armoury | Reference / Inventory | Needs Review |
| CashSnap | Context | Workflow context table for multi-step CashSnap automations | CashSnap | Automation / Context | Current-ish |
| MedRecord | Context 2 | Workflow context table for MedRecord automation processes | MedData | Automation / Context | Needs Refactor |
| CashSnap | [Debt Charges](debt-charges.md) | Interest, fees, penalties, and other non-principal charges applied to debts | CashSnap | Core / Financial | Current |
| CashSnap | debt_mgmt_menu | UI/navigation helper table for debt management views and actions | CashSnap | UI Helper | Needs Review |
| CashSnap | [Debts](debts.md) | Debt accounts including loans, credit cards, installments, personal debts, and MedRecord-created debts | CashSnap | Core / Financial | Current |
| CashSnap | Debug Log | Structured logging for CashSnap bots, workflows, and repair processes | CashSnap | Observability | Current |
| MedRecord | DebugLog | MedRecord-specific logging table | MedData | Observability | Needs Refactor |
| CashSnap | [Ledger](ledger.md) | Financial source of truth for bills, debts, payments, charges, allocations, and adjustments | CashSnap | Core / SSOT | Current |
| MedRecord | Deductible_Tracker | Tracks insurance deductible progress and related medical cost status | MedData | Medical / Insurance / Financial | Needs Review |
| Armoury | Entities | Shared organization/entity records used to model stores, pharmacies, providers, vendors, and other parties | Armoury | Reference / Entity | Needs Refactor |
| Armoury | Entity_Categories | Categorizes entities across domains such as grocery, pharmacy, vendor, provider, or store | Armoury | Reference / Entity | Needs Refactor |
| Armoury | Entity_Roles | Defines role-based relationships for entities across apps and use cases | Armoury | Reference / Entity | Needs Refactor |
| MedRecord | Family_Members | People associated with medical records, expenses, appointments, and insurance context | MedData | Reference / Medical | Needs Review |
| MedRecord | HSA_Accounts | Health Savings Account records and configuration | MedData | Medical / Financial | Needs Review |
| MedRecord | HSA_Transactions | HSA contributions, reimbursements, payments, and transaction activity | MedData | Medical / Financial | Needs Review |
| CashSnap | Income | Recurring or expected income sources | CashSnap | Financial / Planning | Needs Review |
| CashSnap | Income Deposits | Actual income deposit records tied to pay periods or accounts | CashSnap_db_02 | Financial / Planning | Needs Review |
| CashSnap | Installment Schedule | Expected installment payment schedule records for installment-style debts | CashSnap_db_02 | Financial / Planning | Current-ish |
| MedRecord | Insurance_Claims | Insurance claim records, statuses, and claim-related tracking | MedData | Medical / Insurance | Needs Review |
| MedRecord | Invoice_LineItems | Line-item breakdown of medical invoices and charges | MedData | Medical / Financial | Needs Review |
| CashSnap | Loan_Pmt_Splits | Loan payment split instruction rows for principal and interest processing | CashSnap | Workflow / Loan | Current-ish |
| MedRecord | Medical_Expenses | Medical expense records tied to providers, invoices, HSA, insurance, or CashSnap debts | MedData | Medical / Financial | Needs Refactor |
| MedRecord | Medical_Issues | Medical conditions, concerns, diagnoses, or issue tracking | MedData | Medical | Needs Review |
| MedRecord | Medical_Organizations | Medical organizations such as clinics, hospitals, pharmacies, and billing entities | MedData | Reference / Entity / Medical | Needs Refactor |
| MedRecord | Medications | Medication, prescription, dosage, and usage tracking | MedData | Medical / Pharmacy | Needs Review |
| MedRecord | Org_Invoices | Organization-level medical invoices that may create matching CashSnap debts | MedData | Medical / Financial / Cross-App | Needs Refactor |
| MedRecord | Organization_Balances | Aggregated balances owed to medical organizations | MedData | Medical / Financial | Needs Refactor |
| CashSnap | Pay Period | Payroll/pay-period date ranges for dashboard and planning logic | CashSnap_db_02 | Planning | Current-ish |
| CashSnap | Paycheck Selector | Helper table for selecting active pay-period context | CashSnap | UI / Planning | Current-ish |
| CashSnap | [PaymentAllocations](payment-allocations.md) | Debt-side payment split records for principal, interest, fees, and reversals | CashSnap | Core / Financial | Current |
| CashSnap | [Payments](payments.md) | Cash payment records applied to bills or debts | CashSnap | Core / Financial | Current |
| CashSnap | PayPeriod_Generator | Helper/generator table for creating pay-period records | CashSnap_db_02 | Automation / Planning | Needs Review |
| MedRecord | Pharmacy_Fill_Log | Prescription fill, refill, and pharmacy pickup history | MedData | Medical / Pharmacy | Needs Review |
| MedRecord | Provider_Balances | Aggregated balances owed to individual providers | MedData | Medical / Financial | Needs Refactor |
| MedRecord | Provider_Transactions | Transactions tied to providers, invoices, HSA, insurance, or personal payments | MedData | Medical / Financial | Needs Refactor |
| MedRecord | Providers | Directory of doctors, specialists, pharmacies, facilities, or service providers | MedData | Reference / Medical | Needs Refactor |
| CashSnap | [Statements](statements.md) | Statement periods for credit cards and statement-based loans | CashSnap | Core / Financial | Current |
| CashSnap | Summary Totals | Dashboard/reporting summary helper table | CashSnap | Reporting / UI Helper | Needs Review |
| CashSnap | [Transaction Links](transaction-links.md) | Bill-side allocation records linking payments to bill charges | CashSnap | Core / Financial | Current |
| CashSnap | UserSettings | User preferences, configuration, or app-level settings | CashSnap | Configuration | Needs Review |

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
