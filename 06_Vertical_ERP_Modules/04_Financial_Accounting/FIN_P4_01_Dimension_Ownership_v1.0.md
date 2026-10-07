# FIN-P4-01 — Analytical Dimension Ownership (Cost Center / BU)

- **Status:** Accepted
- **Date:** 2026-10-07
- **Module:** Financial Accounting
- **Related:** ADR-FIN-001 §G; Law 5.1; Org `erp_cost_centers` / `erp_business_units`

---

## Decision

| Entity | Owner (SoT) | Finance role |
|--------|-------------|--------------|
| Cost center master | **Organization** (`erp_cost_centers`) | Logical UUID on `fin_acc_journal_items.cost_center_id` |
| Business unit master | **Organization** (`erp_business_units`) | Logical UUID on `fin_acc_journal_items.business_unit_id` |
| Account dimension flags | **Finance** (`fin_acc_accounts.requires_cost_center` / `requires_business_unit`) | Enforce on post / draft validate |
| Fixed asset master + depreciation | **Finance** (`fin_acc_fixed_assets`, `fin_acc_depreciation_runs`) | Depreciation → **journal DRAFT only** (K1 law) |
| Period close checklist | **Finance** | Blockers before soft/hard close (K4) |

## Rules

1. **No duplicate** cost-center or BU tables inside Finance (Law 5.1).
2. No physical FK from Finance to Org tables (Law 2.2) — UUID logical refs only.
3. If account has `requires_cost_center = true` and journal line omits `cost_center_id` → **422**.
4. Same for `requires_business_unit` / `business_unit_id`.
5. Depreciation run never auto-posts; creates suggested or direct **DRAFT** journal for human confirm.
6. Guided period close (K4): checklist must report critical blockers; hard-close blocked while critical open (override only with explicit flag + audit — v1 blocks only).

## Out of scope P4

- Project dimension master
- Full FA disposal workflow UI richness (API + minimal model first)
- Cross-module Org cost-center CRUD from Finance UI (read/consume only)
