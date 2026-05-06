# 🔄 Cross-App Flows

---

## purpose

Defines how data moves between apps.

---

## medrecord-to-cashsnap-debt-flow

<details>
<summary>View Flow</summary>

```text
Appointment
→ Invoice Created
→ Org_Invoice Row
→ Create Debt (CashSnap)
→ Debt tracked in Ledger
```

Key rule:

* CashSnap becomes financial owner

</details>

---

## receipt-to-armoury-flow

<details>
<summary>View Flow</summary>

```text
ReceiptKeeper
→ Receipt Created
→ Armoury Items Added
→ Items linked to receipt
```

Purpose:

* Separate capture from classification

</details>

---

## entity-cross-app-usage

<details>
<summary>View Flow</summary>

```text
Entity (Armoury)
→ Used by:
   - CashSnap (Payees)
   - MedRecord (Providers)
   - ReceiptKeeper (Stores)
```

</details>

---

## rules

<details>
<summary>View Rules</summary>

* One system owns financial truth (CashSnap)
* Other apps feed data into it
* Avoid duplicate financial tracking

</details>

---

## related-docs

* `docs/app-ecosystem/entity-system.md`
