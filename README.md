# 💸 CashSnap Docs

**CashSnap** is a highly automated financial tracking system built with Google AppSheet, designed to manage:

* 📅 Bills (recurring + statement-based)
* 💳 Debts (loans, credit cards, installment plans)
* 💵 Payments and allocations
* 📊 Ledger-driven balances (single source of truth)
* 🧾 Statements and billing cycles
* 🛠️ Debug logging for full traceability

This repository contains the **complete documentation and system design** for CashSnap.

---

# 🧠 Design Philosophy

CashSnap is built around three core principles:

### 1. 📊 Ledger as Source of Truth (SSOT)

All balances and financial state should ultimately be derived from the **Ledger**.

* Bills
* Debts
* Charges
* Payments

Avoid duplicate calculations outside the ledger whenever possible.

---

### 2. 🔄 Event-Driven Automation

The system is powered by AppSheet Bots and Actions triggered by:

* Payment creation/updates
* Statement creation
* Transaction Link creation
* Payment Allocation creation

Automation must be:

* Deterministic
* Idempotent (safe to re-run)
* Carefully gated to avoid duplication

---

### 3. 🧩 Allocation-Based Accounting

Payments are not applied directly — they are **allocated**:

* **Bills → Transaction Links**
* **Debts → PaymentAllocations**

These allocations drive:

* Ledger entries
* Charge fulfillment
* Balance updates

---

# 🏗️ System Architecture

## Core Tables

| Table              | Purpose                                       |
| ------------------ | --------------------------------------------- |
| Bills              | Recurring and statement-based obligations     |
| Bill Charges       | Individual charge instances per billing cycle |
| Payments           | Money paid toward bills/debts                 |
| Transaction Links  | Payment → Bill Charge allocations             |
| Ledger             | Financial source of truth                     |
| Debts              | Loans, credit cards, installment plans        |
| Statements         | Billing periods and summaries                 |
| PaymentAllocations | Debt payment splits (principal/interest/fees) |
| Debt Charges       | Interest/fee obligations                      |
| Debug Log          | System observability                          |

---

# 🔄 Core Workflows

## 💵 New Payment Flow

1. Payment is created
2. Allocation begins:

   * Bills → Transaction Links
   * Debts → PaymentAllocations
3. Allocation triggers:

   * Ledger entries
   * Charge updates
4. Process continues until fully allocated

---

## 🧾 Statement Flow

1. Statement is created
2. Payments within period are attached
3. Allocation process runs
4. Interest/fees become Debt Charges
5. Ledger entries are generated

---

## 🔁 Reversals & Returns

* Reversals create mirrored records:

  * Payments
  * Allocations
  * Ledger entries
* Return Date may differ from original payment period
* System must support cross-statement corrections

---

# 🤖 Automation (Bots)

Key bots include:

* **George** → Handles cleared payments and payoff logic
* **LedgerBot** → Creates ledger entries from Transaction Links
* **Statement Bot** → Processes statement-based flows
* **Loan Statement Bot** → Handles loan-specific allocation logic
* **Payment Repair Bot** → Fixes orphaned or mislinked payments
* **Bill Bot** → Manages bill charge lifecycle

Bots must:

* Avoid duplicate execution
* Use gating fields (`Processed`, `TriggerSource`, etc.)
* Log all major actions

---

# 🧰 Debug Logging

CashSnap uses structured debug logging for full traceability.

Each log includes:

* Process
* Action
* Stage
* Status (`START`, `SUCCESS`, `FAIL`, `INFO`)
* Entity references
* Structured details

### Example

**Message:**

```
Payment | Create TL | SUCCESS | Payment:abc123 | TL Created
```

**Details:**

```
Charge=chg456 | Amt=87.97 | Remaining=0.00
```

This allows:

* Full flow reconstruction
* Failure detection
* Debug dashboards

👉 See: `docs/workflows/debug-logging.md`

---

# ⚠️ Known Challenges

* Duplicate bot execution
* Rounding discrepancies (±0.01)
* Statement vs payment timing mismatches
* Ledger inconsistencies in edge cases
* Debug logging performance overhead

---

# 🚧 Current Focus

* 🔧 Loan Statement process repair
* 🔁 Payment allocation stabilization
* 📊 Ledger consistency and cleanup
* 📅 Bill charge + pay period alignment
* ⚡ Debug log performance optimization

---

# 📚 Documentation Structure

```text
docs/
├─ data-model/       # Table structures and relationships
├─ workflows/        # End-to-end system processes
├─ bots/             # Bot logic and conditions
├─ views/            # AppSheet UX design
└─ formulas/         # Key formulas by table
```

---

# 🤝 Working With This Repo

When making changes to CashSnap:

1. Update relevant docs
2. Update `AI_CONTEXT.md` for major changes
3. Add entries to `CHANGELOG.md`
4. Keep workflows and bots documented

---

# 🧠 For AI / ChatGPT Usage

The file:

```
AI_CONTEXT.md
```

is optimized to:

* Quickly bring AI up to speed
* Reflect current system state
* Improve accuracy of assistance

---

# 🚀 Future Direction

* Full Ledger SSOT implementation
* Self-healing automation flows
* Performance optimization
* Advanced analytics & dashboards

---

# 📌 Notes

* Built entirely in **Google AppSheet**
* Designed for **high automation + traceability**
* Continuously evolving — documentation should reflect current reality
