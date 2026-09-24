# Organization Layer — DDL Notes v1.1 (ORG-DOC-03)

**Status:** Implemented in code (hamarehSaasErp, waves P0–P7)  
**ADR:** ADR-ORG-001  
**Roadmap:** ORG_Layer_Completion_Roadmap_v1.0  
**Date:** 2026-09-24

These notes capture the **as-built** schema surface for the Organization bounded context. They do not replace migrations; they document ownership, RLS, and inter-module rules for architects and integrators.

---

## 1. Ownership & isolation

| Rule | Application |
|------|-------------|
| Tenant isolation | All operational org tables carry `tenant_id`; RLS ENABLE+FORCE under `app_user` |
| Soft delete | `deleted_at` / `deleted_by` on operational tables |
| Concurrency | `row_version BIGINT NOT NULL DEFAULT 1` |
| No physical FK across modules | Logical UUID only (Law 2.2) |
| Master Data SoT | `entity_addresses`, `entity_contact_points` owned by **MasterData** (Law 5.1). Organization must not duplicate. |

---

## 2. Core company spine (`erp_companies`)

| Column (selected) | Notes |
|-------------------|--------|
| `legal_name` | NOT NULL (backfilled from `name` where needed) |
| `trade_name` | Optional commercial name |
| `company_type` / tax / registration fields | P0 enrichment |
| `status` | Legal status; `is_active` retained for BC |
| `is_primary` | One primary per tenant (partial unique) |
| `parent_company_id` | Same-tenant logical parent; cycle guard in service |
| `entity_kind` | `OPERATING` \| `CONSOLIDATION` \| `ELIMINATION` |
| `base_currency_id`, `chart_of_accounts_id`, `default_consol_rate_type` | Logical refs; no cross-module FK |

Related:

- `erp_company_ownerships` — equity/control between companies (same tenant)
- `erp_company_fiscal_assignments` — company ↔ `fin_fiscal_periods.period_id` (logical)
- `erp_company_bank_accounts`, officers — company nested financial/people metadata

---

## 3. Branch & plant (`erp_branches`)

| Column (selected) | Notes |
|-------------------|--------|
| `branch_kind` | e.g. OFFICE, PLANT, WAREHOUSE_SITE |
| Logistics flags | shipping / receiving / manufacturing site |
| `parent_branch_id` | Hierarchy within company; cycle guard |
| `default_warehouse_id` | Logical inventory ref |
| `erp_branch_warehouse_maps` | Many warehouses per branch |

Departments: `company_id` on departments for list-under-company; still linked to branch.

---

## 4. Management dimensions

| Table | Purpose |
|-------|---------|
| `erp_business_units` + company map | BU independent of legal tree |
| `erp_cost_centers` | Cost dimension under company |
| `erp_org_hierarchies` + `erp_org_hierarchy_nodes` | Multi-purpose trees (LEGAL, MANAGEMENT, TAX, …) |
| Sales / Purchasing org + assignments | Org boundary only; no order engine |
| IC partners / rules | Mapping + mirror policy; posting deferred |
| Consolidation runs | Snapshot of hierarchy nodes; accounting engine deferred |

---

## 5. MasterData polymorphic entities (shared)

| Table | Filter for company |
|-------|--------------------|
| `entity_addresses` | `entity_type='COMPANY'`, `entity_id=<company_id>` |
| `entity_contact_points` | same |

API: `/api/master-data/entity-addresses|entity-contact-points`  
Index supports query filters `entity_type`, `entity_id`.  
`address_type_id` / `country_id` may be null when UI has no catalog yet.

---

## 6. HTTP surface (Organization module prefix)

Prefix (module provider): `/api/…/organization`

Notable resources: companies, branches, departments, bank-accounts, officers, cost-centers, ownerships, fiscal-assignments, business-units, hierarchies(+nodes), intercompany, sales-organizations, purchasing-organizations, consolidation-runs, `POST structure/apply-template`.

Permissions: `organization.*.view|manage|create|update|delete` family (see PermissionSeeder). New codes require seed + re-login.

---

## 7. Sample data

Seeder: `AriaSanatDemoOrgSeeder`

```bash
docker compose exec app php artisan db:seed --class=PermissionSeeder
docker compose exec app php artisan db:seed --class=DemoTenantOwnerSeeder
docker compose exec app php artisan db:seed --class=AriaSanatDemoOrgSeeder
```

Creates ARYA-HQ (primary), ARYA-SUB (parent HQ, 80% ownership), HQ branch, BU, LEGAL hierarchy nodes.

---

## 8. Explicit non-goals (v1.1)

- Physical FK to Accounting / Inventory tables  
- IC document posting engine  
- Full consol worksheet / rate engine  
- Duplicate address tables inside Organization  

---

## 9. Change log

| Version | Summary |
|---------|---------|
| v1.0 | Roadmap + ADR-ORG-001 |
| **v1.1** | As-built DDL notes after P0–P7 implementation |
