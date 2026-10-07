# FIN-P0-01 — Ownership Decision: Finance vs Org (company / fiscal / dimensions)

- **Status:** Accepted
- **Date:** 2026-10-07
- **Parent:** ADR-FIN-001 §5; ADD-07; Law 5.1 Master Data SoT
- **Code module:** `App\Modules\FinancialAccounting`

---

## 1. Purpose

Lock which tables Finance **owns** vs **consumes** so P0 migrations do not duplicate Org fiscal/company masters.

---

## 2. Decision summary

| Concern | Owner module | Table / artifact | Finance usage |
|---------|--------------|------------------|---------------|
| Legal company master | Organization | `erp_companies` | Logical `company_id` UUID on every journal + ledger |
| Company ↔ period assignment | Organization | `erp_company_fiscal_assignments` | Read-only validation that company is assigned to period |
| Cost center / BU | Organization | `erp_cost_centers`, `erp_business_units` | Logical UUID on journal lines (nullable in P0; required rules in P4) |
| Company bank accounts | Organization | `erp_company_bank_accounts` | Consumed in P1 Treasury (no Finance duplicate) |
| Fiscal calendar (year/period dates) | Platform / existing `fin_fiscal_periods` | `fin_fiscal_periods` (legacy Accounting table; period_id) | Logical `period_id` on journals + period_controls; **do not recreate calendar** |
| Posting lock state per company+period | **FinancialAccounting** | `fin_acc_period_controls` | States: `OPEN` / `SOFT_CLOSED` / `HARD_CLOSED` |
| Leading ledger per company | **FinancialAccounting** | `fin_acc_ledgers` | One leading ledger active in P0 |
| Chart of accounts | **FinancialAccounting** | `fin_acc_accounts` | Tree; unique `(tenant_id, account_code)` soft-delete-aware |
| Journal header/lines | **FinancialAccounting** | `fin_acc_journal_entries`, `fin_acc_journal_items` | Double-entry; NUMERIC(20,4); reverse-only after post |
| Document number sequences | **FinancialAccounting** | `fin_acc_document_sequences` | Per tenant+company+fiscal_year gap-safe |

---

## 3. Fiscal period policy (critical)

1. **Calendar SoT** remains `fin_fiscal_periods` (name, start_date, end_date). Organization already links companies via `erp_company_fiscal_assignments.period_id` (logical, no physical FK).
2. Finance **does not** introduce a second fiscal year/period master.
3. Finance **owns** operational control via `fin_acc_period_controls`:
   - Key: `(tenant_id, company_id, period_id)` unique
   - `control_status`: `OPEN` | `SOFT_CLOSED` | `HARD_CLOSED`
   - `HARD_CLOSED` blocks `post` and uncontrolled `reverse`
   - Soft-close may still allow controlled reverse/adjustment per product rules later
4. Legacy `fin_fiscal_periods.is_closed` is **not** the GL posting gate; Finance control table is authoritative for journal posting.

---

## 4. Company & CoA binding

- `erp_companies.chart_of_accounts_id` is a **logical** hook (already on Company). In P0, CoA lives in `fin_acc_accounts` scoped by `tenant_id` (and optionally company_id policy).
- P0 policy: **tenant-wide CoA** with mandatory `company_id` on journal header (multi-company books, shared account tree unless product later sells company-specific CoA pack).
- `fin_acc_ledgers.company_id` + `is_leading` ensure one leading ledger per company.

---

## 5. Explicit non-goals for P0

- No new `fin_acc_cost_centers` (supersedes old sketch in `fin_accounting_table_definitions.md` that duplicated Org).
- No Finance-owned bank account master.
- No physical FK to Org tables (Law 2.2).
- Legacy module `App\Modules\Accounting` remains untouched in this wave; new code path is `FinancialAccounting` only. Migration of legacy data is out of P0 scope.

---

## 6. Acceptance (FIN-P0-01)

- [x] Document states what Finance owns vs consumes
- [x] No duplicate fiscal calendar in Finance migrations
- [x] Period posting states owned by `fin_acc_period_controls`
- [x] Cost center / bank / company masters stay in Org

---

## 7. Follow-ups

- Amend `fin_accounting_table_definitions.md` in a later docs commit to mark cost-center table as superseded by Org.
- When implementing P0-05 migration, create only `fin_acc_period_controls`, not a second periods table.
