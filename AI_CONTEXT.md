# CashSnap AI Context

## 🧠 System Overview

CashSnap is a Google AppSheet-based financial management system designed to track:

* Bills (recurring + statement-based)
* Debts (multiple models: loan, credit card, installment)
* Payments and allocations
* Ledger (intended single source of truth)
* Statements and billing cycles

The system emphasizes automation via Bots, Processes, and Actions, with Debug Logs used for traceability.

---

## 🧱 Core Tables

* Bills
* Bill Charges
* Payments
* Transaction Links
* Ledger
* Debts
* Statements
* PaymentAllocations
* Debt Charges
* Debug Log

---

## 🧭 Current Architecture Direction

### 🔹 Ledger as Source of Truth (SSOT)

All balances should ultimately derive from Ledger:

* Bill balances
* Debt balances
* Charge balances
* Payment effects

Avoid duplicate calculations outside Ledger when possible.

---

### 🔹 Event-Driven System

Flows are triggered by:

* Payment creation/update
* Statement creation
* Transaction Link creation
* PaymentAllocation creation

Bots must be carefully gated to avoid duplicate execution.

---

### 🔹 Allocation-Based Model

Payments are broken into:

* Transaction Links (Bills)
* PaymentAllocations (Debts)

These drive:

* Ledger entries
* Charge fulfillment
* Balance updates

---

## 🔄 Current Priority Work

### 1. Loan Statement Process Repair

* Loan_Pmt_Splits define allocation instructions
* PaymentAllocations are the execution layer
* Debt Charges represent interest/fees
* Ledger reflects all financial impact

Goal:

* Remove duplicate charge creation
* Ensure 1:1 mapping of allocations → ledger

---

### 2. Payment Allocation Stabilization

* Prevent duplicate bot execution
* Ensure deterministic allocation order
* Fix edge cases (future charges, partial allocations)

---

### 3. Ledger Consistency

* Standardize Signed Amount logic
* Ensure all financial events create ledger entries
* Eliminate mismatches between:

  * Ledger
  * Charges
  * Payments

---

### 4. Bill Charge + Pay Period Logic

* Improve cycle detection
* Fix circular dependencies
* Align with Pay Period dashboard

---

### 5. Debug Log Optimization

* Maintain structured logs
* Reduce performance overhead
* Preserve traceability

Reference:
See Debug Logging design in docs/workflows/debug-logging.md 

---

## 🧪 Known Issues

* Bots sometimes fire multiple times
* Rounding discrepancies (±0.01)
* Statement vs payment timing mismatches
* Debug logs may slow sync performance
* Some ledger entries missing or duplicated

---

## 🧰 Debug Logging Standard

Columns:

* Timestamp
* Process
* Action
* Stage
* Status (START / SUCCESS / FAIL / INFO)
* Source Table
* Message
* Details
* TraceID
* Row Identifier
* Bill ID
* Debt ID
* TransLink ID
* Ledger ID
* ProcessRunID

### Message Format

[Process] | [Stage] | [Status] | [SourceTable]:[RowID] | Summary

### Details Format

Key=Value | Key=Value | Key=Value

---

## ⚠️ AppSheet Constraints to Remember

* ROUND() only accepts 1 argument
* SELECT() is expensive — minimize repetition
* Avoid circular dependencies
* Type consistency matters (Price vs Decimal)
* Bots can re-trigger on updates unintentionally

---

## 🧠 How ChatGPT Should Assist

Assume:

* System is mid-refactor (not “clean slate”)
* Backward compatibility matters
* Debug visibility is critical
* Automation must be deterministic
* Ledger integrity is highest priority

Prefer:

* Step-by-step flows over theory
* Explicit formulas and conditions
* Bot-safe logic (idempotent where possible)
* Minimal performance overhead

---

## 📌 When Updating This File

Always update when:

* Schema changes
* New core logic introduced
* Bots redesigned
* Major bugs identified/resolved
