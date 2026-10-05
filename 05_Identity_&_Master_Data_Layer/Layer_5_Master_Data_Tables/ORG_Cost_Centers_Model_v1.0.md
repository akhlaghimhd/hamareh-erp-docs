# ORG Cost Centers — Complete Model v1.0

**Status:** Accepted for Organization master data  
**Owner module:** Organization (SoT = `erp_cost_centers`)  
**Date:** 2026-10-06  
**Related:** ADR-ORG-001, ORG Layer Completion Roadmap, Law 5.1 Master Data

---

## 1. Purpose

A **Cost Center (مرکز هزینه)** is a *management / controlling dimension*, not a physical org unit.

It answers: **where was cost incurred and who is accountable?**

Used later by Accounting, Procurement, Payroll, Fixed Assets, and management reports.  
Until the Accounting module is live, Organization only owns the **master catalog**.

---

## 2. Product rules

| Rule | Detail |
|------|--------|
| Scope | Always **company-scoped** (`company_id` required) |
| Hierarchy | Optional parent tree; parent must be same company; **no cycles** |
| Department | Optional link (`department_id`); department ≠ cost center |
| Code uniqueness | Unique per `(tenant_id, company_id, code)` among non-deleted rows |
| Soft delete | Law 1.4 — no hard delete from tenant UI |
| Isolation | `tenant_id` + PostgreSQL RLS FORCE |
| Cross-module FK | **Forbidden** — consumers use logical UUID only (Law 2.2) |

---

## 3. Canonical data model (`erp_cost_centers`)

### 3.1 Columns (v1.0 complete master)

| Column | Type | Required | Notes |
|--------|------|----------|-------|
| `cost_center_id` | UUID PK | yes | |
| `tenant_id` | UUID | yes | RLS |
| `company_id` | UUID | yes | logical → erp_companies |
| `department_id` | UUID NULL | no | logical → departments |
| `parent_cost_center_id` | UUID NULL | no | self tree, same company |
| `code` | VARCHAR(50) | yes | unique per company (soft) |
| `name` | VARCHAR(200) | yes | |
| `cost_center_type` | VARCHAR(30) | yes | see enum; default `ADMIN` |
| `manager_user_id` | UUID NULL | no | logical → Identity tenant user |
| `description` | VARCHAR(500) NULL | no | free text |
| `valid_from` | DATE NULL | no | inclusive |
| `valid_to` | DATE NULL | no | inclusive; ≥ valid_from when both set |
| `is_active` | BOOLEAN | yes | default true |
| audit + soft delete + `row_version` | standard | yes | Law 1.4 / 1.5 |

### 3.2 `cost_center_type` enum

| Code | FA label | Typical use |
|------|----------|-------------|
| `ADMIN` | اداری | HQ, finance, HR overhead |
| `SALES` | فروش | sales teams / regions |
| `PRODUCTION` | تولید | plant / line |
| `SUPPORT` | پشتیبانی | IT, facilities |
| `R_AND_D` | تحقیق و توسعه | R&D projects overhead |
| `SHARED` | مشترک / تسهیم | shared service to allocate later |
| `OTHER` | سایر | catch-all |

### 3.3 Indexes

- `uq_erp_cc_code` on `(tenant_id, company_id, code) WHERE deleted_at IS NULL`
- `idx_erp_cc_company` on `(tenant_id, company_id)`
- `idx_erp_cc_parent` on `(tenant_id, parent_cost_center_id)`
- `idx_erp_cc_type` on `(tenant_id, company_id, cost_center_type)` (optional)

---

## 4. API surface (Organization)

| Method | Path | Permission |
|--------|------|------------|
| GET | `/api/v1/organization/companies/{company}/cost-centers` | `organization.cost_center.view` |
| POST | `/api/v1/organization/companies/{company}/cost-centers` | `organization.cost_center.manage` |
| PUT | `/api/v1/organization/cost-centers/{id}` | `organization.cost_center.manage` |
| DELETE | `/api/v1/organization/cost-centers/{id}` | `organization.cost_center.manage` |

Validation: code/name required on create; type ∈ enum; valid_to ≥ valid_from; parent same company + no cycle; code unique per company.

---

## 5. UI (Company detail)

Collapsible **مراکز هزینه**: table shows code, name, type, active, actions.  
Form: code*, name*, type*, parent, description, valid_from/to (Shamsi), is_active.

---

## 6. Explicit debt — Accounting & consumers (NOT in this delivery)

**ORG-CC-DEBT** until Accounting / consumers land:

| ID | Item | Owner later |
|----|------|-------------|
| CC-D1 | `cost_center_id` on journal entry lines | Accounting |
| CC-D2 | Mandatory/optional posting rules by account or doc type | Accounting |
| CC-D3 | Annual/monthly budget per CC + variance | Accounting / Controlling |
| CC-D4 | Allocation cycles (SHARED → receivers) | Controlling |
| CC-D5 | Link to Profit Center | Accounting / Org |
| CC-D6 | Payroll distribution by cost center | HR / Payroll |
| CC-D7 | PO / purchase invoice line cost center | Purchasing |
| CC-D8 | Fixed-asset depreciation target CC | Assets |
| CC-D9 | Analytic reports (P&L by CC, tree roll-up) | Reporting |
| CC-D10 | Retention/purge UX for deleted CC bucket | Platform / Org UI |

> Do not invent parallel cost-center tables in other modules (Law 5.1 / 2.2).

---

## 7. Out of scope v1.0

- Budget amounts and fiscal-year versions
- Statistical key figures / activity types
- Cross-company cost centers

---

## 8. Changelog

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-10-06 | Complete master model; accounting debt locked |
