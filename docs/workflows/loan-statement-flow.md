# 🧾 Loan Statement Flow

## 🎯 Purpose

This workflow processes **Loan (Statement-Based)** debts by:

* Attaching payments to a statement period
* Splitting payments into **Interest / Principal (and Fees if applicable)**
* Creating **Debt Charges** for interest/fees
* Generating **Ledger entries** for all financial impact

---

# 🧠 Core Design Principles

### 1. Statement Drives Allocation

* Statements define the **time window**
* Payments are attached to a statement
* Allocations are derived from **Loan_Pmt_Splits**

---

### 2. Separation of Roles

| Component          | Responsibility                         |
| ------------------ | -------------------------------------- |
| Loan_Pmt_Splits    | Instruction layer (what should happen) |
| PaymentAllocations | Execution layer (what did happen)      |
| Debt Charges       | Represents interest/fees owed          |
| Ledger             | Financial source of truth              |

---

### 3. No Duplication Rule

* Interest/Fee charges must be created **exactly once**
* PaymentAllocations must map **1:1 to ledger entries**
* No duplicate automation paths allowed

---

# 🏗️ High-Level Flow

```text
Add Statement
→ Loan Statement Bot triggers
→ Attach Payments to Statement
→ Create Context (optional but recommended)
→ Process Loan_Pmt_Splits
→ Create PaymentAllocations
→ Create / Link Debt Charges (Interest/Fee)
→ Create Ledger Entries
→ Lock Context / Mark Process Complete
```

---

# 🔄 Detailed Step-by-Step Flow

## 🔹 Step 1 — Statement Created

Trigger:

* New row added to **Statements**

Condition:

```appsheet
[Debt_ID].[Model] = "Loan (Statement-Based)"
```

---

## 🔹 Step 2 — Loan Statement Bot Starts

### Debug Log (START)

```
LoanStatement | Start | START | Statement:[StatementID] | Processing loan statement
```

---

## 🔹 Step 3 — Attach Payments to Statement

Find all payments:

```appsheet
SELECT(
  Payments[Row ID],
  AND(
    [Debt ID] = [_THISROW].[Debt_ID],
    [Date] >= [_THISROW].[Statement_Start],
    [Date] <= [_THISROW].[Statement_End]
  )
)
```

Update:

* Set `[Statement_ID]` on each payment

---

### Debug Log

```
LoanStatement | Attach Payments | SUCCESS | Statement:[StatementID] | Payments attached
```

---

## 🔹 Step 4 — Create Processing Context (Recommended)

Create a Context row with:

* StatementID
* DebtID
* ProcessRunID / TraceID
* Locked = FALSE

Purpose:

* Prevent duplicate runs
* Group all related actions

---

## 🔹 Step 5 — Process Loan_Pmt_Splits

Each split row represents:

| Field         | Meaning             |
| ------------- | ------------------- |
| Payment       | Which payment       |
| Interest      | Interest portion    |
| Principal     | Principal portion   |
| Total Payment | Full payment amount |

---

### ⚠️ Rule

Loan_Pmt_Splits:

* ❌ DO NOT create Debt Charges directly
* ✅ ONLY define allocation instructions

---

## 🔹 Step 6 — Create PaymentAllocations

For each split:

### Create 2 rows:

1. Interest Allocation
2. Principal Allocation

---

### Interest Allocation

| Field          | Value      |
| -------------- | ---------- |
| AllocationType | "Interest" |
| Amount         | [Interest] |
| PaymentID      | Payment    |
| StatementID    | Statement  |
| DebtID         | Debt       |

---

### Principal Allocation

| Field          | Value       |
| -------------- | ----------- |
| AllocationType | "Principal" |
| Amount         | [Principal] |
| PaymentID      | Payment     |
| StatementID    | Statement   |
| DebtID         | Debt        |

---

### Debug Log

```
LoanStatement | Create Allocations | SUCCESS | Payment:[PaymentID] | Interest & Principal allocations created
```

---

## 🔹 Step 7 — Interest/Fee Charge Handling

For each **Interest Allocation**:

### Find existing charge:

```appsheet
SELECT(
  Debt Charges[Row ID],
  AND(
    [Debt ID] = [_THISROW].[DebtID],
    [ChargeType] = "Interest",
    [StatementID] = [_THISROW].[StatementID]
  )
)
```

---

### If NOT FOUND:

Create new Debt Charge:

* ChargeType = Interest
* Amount = allocation amount
* Link to Statement

---

### If FOUND:

* Use existing charge

---

### Link:

Set `[Debt Charge ID]` on PaymentAllocation

---

### Debug Log

```
LoanStatement | Interest Charge | SUCCESS | PaymentAlloc:[AllocID] | Charge linked/created
```

---

## 🔹 Step 8 — Ledger Entry Creation

### Responsibility:

Handled by **Pmt Allocation Bot** (NOT Statement Bot)

---

### Interest Allocation → 2 Ledger Effects

1. **+ Interest Charge**

   * Created when Debt Charge is created

2. **- Interest Payment**

   * Created from PaymentAllocation

---

### Principal Allocation → 1 Ledger Effect

* **- Principal Applied**

---

### Ledger Requirements

Each row should include:

* Debt ID
* PaymentID
* StatementID
* PmtAllocID
* Debt Charge ID (interest only)
* Allocation Type
* Signed Amount

---

### Debug Logs

```
Ledger | Create | START | Alloc:[AllocID] | Creating ledger entry
```

```
Ledger | Create | SUCCESS | Alloc:[AllocID] | Ledger entry created
```

---

## 🔹 Step 9 — Prevent Duplicate Ledger

⚠️ Critical Rule:

* Full Payment ledger rows MUST NOT also reduce balance if split allocations exist

Options:

* Delete old rows
* OR set `[Affects Balance?] = FALSE`

---

## 🔹 Step 10 — Lock Context / Complete Process

Update Context:

* Locked = TRUE
* Process Complete = TRUE

---

### Debug Log

```
LoanStatement | Complete | SUCCESS | Statement:[StatementID] | Processing complete
```

---

# 📊 Expected Outcome (Per Payment)

For each payment:

| Component          | Count                    |
| ------------------ | ------------------------ |
| PaymentAllocations | 2 (Interest + Principal) |
| Debt Charges       | 1 (Interest)             |
| Ledger Entries     | 3                        |
|                    | + Interest Charge        |
|                    | - Interest Payment       |
|                    | - Principal Applied      |

---

# ⚠️ Common Failure Points

### ❌ Duplicate Interest Charges

Cause:

* Created by both Loan_Pmt_Splits AND PaymentAllocations

Fix:

* Only PaymentAllocations create charges

---

### ❌ Missing Ledger Entries

Cause:

* Allocation bot not triggered or mis-gated

---

### ❌ Principal Reducing Twice

Cause:

* Full payment ledger + allocation ledger both active

Fix:

* Disable full payment ledger impact

---

### ❌ Payment Not Attached to Statement

Cause:

* Payment Repair Bot not run or misconfigured

---

# 🧪 Testing Strategy

### Test 1 Payment Only

Expect:

* 1 Statement
* 1 Payment
* 2 Allocations
* 1 Interest Charge
* 3 Ledger rows

---

### Validate:

* No duplicate charges
* Principal reduced correctly
* Interest fully paid (if applicable)
* Ledger matches balances

---

# 🧠 Design Notes

* This system is **mid-refactor**
* Backward compatibility must be considered
* Ledger integrity is top priority
* Debug logs are essential for tracing

---

# 🔗 Related Docs

* `docs/workflows/payment-allocation-flow.md`
* `docs/workflows/debug-logging.md`
* `docs/data-model/payment-allocations.md`
* `docs/data-model/debt-charges.md`
