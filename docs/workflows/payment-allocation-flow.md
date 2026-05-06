# 💵 Payment Allocation Flow

## 🎯 Purpose

The Payment Allocation flow controls how debt payments are split and applied across:

* Principal
* Interest
* Fees

This workflow is especially important for:

* Loan (Statement-Based) debts
* Loan (Transaction-Based) debts
* Installment-style debts
* Reversals / returned payments

---

# 🧠 Core Design Principle

A **Payment** represents cash movement.

A **PaymentAllocation** represents how that cash is applied.

The **Ledger** records the financial effect.

```text
Payment
→ PaymentAllocation
→ Debt Charge / Ledger
→ Debt Balance
```

---

# 🧱 Table Responsibilities

| Table              | Responsibility                                    |
| ------------------ | ------------------------------------------------- |
| Payments           | Records cash payment or reversal                  |
| PaymentAllocations | Splits payment into principal / interest / fee    |
| Debt Charges       | Represents interest or fee owed                   |
| Ledger             | Source of truth for financial impact              |
| Statements         | Optional statement-period context                 |
| Loan_Pmt_Splits    | Instruction/source rows for statement-based loans |

---

# 🔄 High-Level Flow

```text
PaymentAllocation Added
→ Determine AllocationType
→ Branch by Principal / Interest / Fee
→ Create or link Debt Charge if needed
→ Create Ledger row
→ Mark allocation processed
→ Log result
```

---

# 🧩 Allocation Types

## Principal

Principal allocations:

* Reduce loan principal
* Do **not** create Debt Charges
* Create one Ledger row

Ledger row:

```text
Type = Principal Applied
Signed Amount = negative amount
Debt Charge ID = blank
Allocation Type = Principal
```

---

## Interest

Interest allocations:

* Apply payment toward interest owed
* Must link to an Interest Debt Charge
* Create payment-side Ledger row

Ledger row:

```text
Type = Debt Payment
Charge Category = Interest
Allocation Type = Interest
Debt Charge ID = linked Interest Debt Charge
Signed Amount = negative amount
```

Interest Debt Charge creation should create its own positive Ledger row separately.

---

## Fee

Fee allocations:

* Apply payment toward fee owed
* Must link to a Fee Debt Charge
* Create payment-side Ledger row

Ledger row:

```text
Type = Debt Payment
Charge Category = Fee
Allocation Type = Fee
Debt Charge ID = linked Fee Debt Charge
Signed Amount = negative amount
```

Fee Debt Charge creation should create its own positive Ledger row separately.

---

# 🤖 Bot Ownership

## Pmt Allocation Bot

Trigger:

```appsheet
Adds only on PaymentAllocations
```

Recommended condition:

```appsheet
AND(
  [Processed?] <> TRUE,
  ISNOTBLANK([PaymentID]),
  ISNOTBLANK([DebtID]),
  [Amount] > 0
)
```

The bot should own:

* Detecting allocation type
* Creating/linking Debt Charges for Interest/Fee
* Creating allocation-side Ledger rows
* Marking PaymentAllocation as processed
* Logging START / SUCCESS / FAIL

---

# 🔹 Branch 1 — Principal Allocation

Condition:

```appsheet
[AllocationType] = "Principal"
```

Actions:

1. Create Ledger row
2. Link Ledger ID back to PaymentAllocation if applicable
3. Mark allocation processed
4. Write Debug Log

Ledger values:

```text
Debt ID = [DebtID]
PaymentID = [PaymentID]
StatementID = [StatementID]
PmtAllocID = [Row ID]
Type = Principal Applied
Allocation Type = Principal
Amount = [Amount]
Signed Amount = negative
Affects Balance? = TRUE
Debt Charge ID = blank
```

Debug message:

```text
PmtAllocation | Principal Ledger | SUCCESS | PaymentAlloc:[Row ID] | Principal applied
```

---

# 🔹 Branch 2 — Interest Allocation

Condition:

```appsheet
[AllocationType] = "Interest"
```

Actions:

1. Find existing Interest Debt Charge
2. Create one if missing
3. Link Debt Charge ID to PaymentAllocation
4. Create Interest Payment Ledger row
5. Mark allocation processed
6. Write Debug Log

Find matching charge:

```appsheet
ANY(
  SELECT(
    Debt Charges[Row ID],
    AND(
      [Debt ID] = [_THISROW].[DebtID],
      [ChargeType] = "Interest",
      [StatementID] = [_THISROW].[StatementID],
      [Amount] = [_THISROW].[Amount]
    )
  )
)
```

Ledger values:

```text
Debt ID = [DebtID]
PaymentID = [PaymentID]
StatementID = [StatementID]
PmtAllocID = [Row ID]
Debt Charge ID = linked Interest Debt Charge
Type = Debt Payment
Charge Category = Interest
Allocation Type = Interest
Amount = [Amount]
Signed Amount = negative
Affects Balance? = TRUE
```

Debug message:

```text
PmtAllocation | Interest Ledger | SUCCESS | PaymentAlloc:[Row ID] | Interest payment applied
```

---

# 🔹 Branch 3 — Fee Allocation

Condition:

```appsheet
[AllocationType] = "Fee"
```

Actions:

1. Find existing Fee Debt Charge
2. Create one if missing
3. Link Debt Charge ID to PaymentAllocation
4. Create Fee Payment Ledger row
5. Mark allocation processed
6. Write Debug Log

Ledger values:

```text
Debt ID = [DebtID]
PaymentID = [PaymentID]
StatementID = [StatementID]
PmtAllocID = [Row ID]
Debt Charge ID = linked Fee Debt Charge
Type = Debt Payment
Charge Category = Fee
Allocation Type = Fee
Amount = [Amount]
Signed Amount = negative
Affects Balance? = TRUE
```

Debug message:

```text
PmtAllocation | Fee Ledger | SUCCESS | PaymentAlloc:[Row ID] | Fee payment applied
```

---

# 🧾 Debt Charge Rules

Debt Charges should exist for:

* Interest
* Fees
* Penalties
* Other non-principal charges

Debt Charges should **not** normally exist for:

* Principal

When a Debt Charge is added, a separate Debt Charge bot should create the positive Ledger row.

Example:

```text
Debt Charge Added
→ Ledger row: + Interest Charge
```

Ledger values:

```text
Type = Debt Charge
Charge Category = Interest/Fee
Debt Charge ID = [Debt Charge Row ID]
Amount = [Amount]
Signed Amount = positive
Affects Balance? = TRUE
```

---

# 🔁 Reversal Handling

A reversal payment should create mirrored allocations.

Original payment:

```text
Interest Allocation = 43.75
Principal Allocation = 24.78
```

Reversal payment:

```text
Interest Reversal Allocation = 43.75
Principal Reversal Allocation = 24.78
```

Recommended fields:

```text
Is Reversal? = TRUE
Original Allocation ID = original PaymentAllocation row
Reversal PaymentID = reversal payment
```

Reversal ledger effects:

| Allocation Type | Ledger Type           | Signed Amount |
| --------------- | --------------------- | ------------- |
| Principal       | Principal Reversal    | Positive      |
| Interest        | Debt Payment Reversal | Positive      |
| Fee             | Debt Payment Reversal | Positive      |

---

# ⚠️ Duplicate Prevention

Each PaymentAllocation should create only one ledger row.

Recommended helper column:

```text
Related Ledger Count
```

Formula:

```appsheet
COUNT(
  SELECT(
    Ledger[LedgerID],
    [PmtAllocID] = [_THISROW].[Row ID]
  )
)
```

Bot condition should include:

```appsheet
[Related Ledger Count] = 0
```

Final recommended bot condition:

```appsheet
AND(
  [Processed?] <> TRUE,
  [Related Ledger Count] = 0,
  ISNOTBLANK([PaymentID]),
  ISNOTBLANK([DebtID]),
  [Amount] > 0
)
```

---

# 🧪 Expected Results

## Principal Allocation

Expected:

* 1 PaymentAllocation
* 0 Debt Charges
* 1 Ledger row

---

## Interest Allocation

Expected:

* 1 PaymentAllocation
* 1 linked Debt Charge
* 1 positive Debt Charge Ledger
* 1 negative Debt Payment Ledger

---

## Fee Allocation

Expected:

* 1 PaymentAllocation
* 1 linked Debt Charge
* 1 positive Debt Charge Ledger
* 1 negative Debt Payment Ledger

---

# 🧠 Balance Impact

## Principal Balance

Should be reduced only by:

```text
Principal Applied ledger rows
```

Not by:

```text
Interest payments
Fee payments
Full payment rows if split allocations exist
```

---

## Interest Balance

Should be:

```text
Interest Charges - Interest Payments
```

---

## Fee Balance

Should be:

```text
Fee Charges - Fee Payments
```

---

## Remaining Balance

For statement-based loans:

```text
Principal Balance + Interest Balance + Fee Balance
```

---

# 🧰 Debug Logging

Use the current CashSnap Debug Log standard.

## START

```text
PmtAllocation | Process | START | PaymentAlloc:[Row ID] | Allocation processing started
```

## SUCCESS

```text
PmtAllocation | Process | SUCCESS | PaymentAlloc:[Row ID] | Allocation processing complete
```

## FAIL

```text
PmtAllocation | Process | FAIL | PaymentAlloc:[Row ID] | Missing required field
```

## Details

```text
PaymentID=[PaymentID] | DebtID=[DebtID] | StatementID=[StatementID] | Type=[AllocationType] | Amt=[Amount] | TraceID=[TraceID]
```

---

# 🚫 Anti-Patterns

Avoid:

* Creating Interest Debt Charges directly from Loan_Pmt_Splits
* Creating full Debt Payment ledger rows when split allocations exist
* Letting Payments and PaymentAllocations both reduce principal
* Using total payment amount as principal reduction
* Creating Debt Charges for principal unless intentionally modeling principal installments as charges

---

# ✅ Implementation Checklist

* [ ] PaymentAllocations has `Processed?`
* [ ] PaymentAllocations has `Debt Charge ID`
* [ ] PaymentAllocations has `StatementID`
* [ ] PaymentAllocations has `PmtAllocID` or key available to Ledger
* [ ] Ledger has `PmtAllocID`
* [ ] Ledger has `Allocation Type`
* [ ] Ledger has `StatementID`
* [ ] Ledger has `Debt Charge ID`
* [ ] Interest/Fee allocations link to Debt Charges
* [ ] Principal allocations do not create Debt Charges
* [ ] Old full-payment ledger rows are disabled or excluded
* [ ] Debug Logs are created at START / SUCCESS / FAIL

---

# 🔗 Related Docs

* `docs/workflows/loan-statement-flow.md`
* `docs/workflows/reversals-and-returns.md`
* `docs/data-model/payment-allocations.md`
* `docs/data-model/debt-charges.md`
* `docs/formulas/debts.md`
