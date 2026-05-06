# 🧩 Entity System

---

## purpose

Defines shared representation of organizations across all apps.

---

## core-problem

Same organization appears as:

* Grocery store (CashSnap)
* Pharmacy (MedRecord)
* Vendor (Armoury)

---

## design-goal

```text
One entity
→ Multiple roles
→ Multiple systems
```

---

## current-structure

<details>
<summary>View Current Tables</summary>

* Entities
* Entity_Categories
* Entity_Roles

</details>

---

## problems

<details>
<summary>View Problems</summary>

* Duplicate entities
* Weak role modeling
* No enforced cross-app linking

</details>

---

## target-design

<details>
<summary>View Target Model</summary>

```text
Entity
→ Roles (Pharmacy, Grocery, Provider)
→ Linked to:
   - Debts
   - Providers
   - Receipts
```

</details>

---

## future-work

<details>
<summary>View Plan</summary>

* Normalize entity usage across apps
* Replace duplicate tables
* Add role-based logic

</details>

---

## related-docs

* `docs/app-ecosystem/cross-app-flows.md`
