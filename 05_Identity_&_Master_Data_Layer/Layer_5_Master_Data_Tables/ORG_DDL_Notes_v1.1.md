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

**UI copy (FE):** field label «نقش شرکت در گروه»; options use plain language (شرکت عملیاتی / سطح تجمیع گروه / حذف معاملات درون‌گروه) plus helper text. Values stored remain the English codes above.

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

## 9. Commercial feature packs (platform / SaaS Admin) — product law

**Status:** Accepted product decision (2026-09-25). Implementation of flag enforcement is platform-owned; Organization module must be ready to respect flags on every surface.

### 9.1 Product intent

Capabilities such as multi-company, multi-branch, intercompany, consolidation entity kinds, and advanced hierarchy are **sellable per tenant**. A tenant that did **not** purchase multi-company / multi-branch must still use the rest of Organization (departments, cost centers, officers, bank accounts, etc.) without being forced through multi-entity UX.

### 9.2 Onboarding rule (platform owner)

At tenant provisioning / onboarding, the **platform owner** (not the tenant end-user) creates:

1. The **primary operating company** (already required by product).
2. A **hidden default HQ branch** under that company (implicit single site).

Downstream entities (departments and later logistics/finance scopes) attach to that default branch. When `org.multi_branch` is OFF, branch list/create is hidden or read-only; the default branch remains in data so models stay consistent.

### 9.3 Gating rule (all Organization surfaces)

Every Organization hub card, list page, create/edit flow, and related API entry path **must** be show/hide or allow/deny based on the tenant’s purchased feature list. Do not hard-code plan logic inside domain services; resolve flags from SaaS Platform / SaaS Admin (subscription or feature flags).

### 9.4 Suggested flag keys (draft)

| Flag | Effect when OFF |
|------|------------------|
| `org.multi_company` | No second company; hide companies list create, parent/ownership, multi-company hub paths; keep primary company usable |
| `org.multi_branch` | No branch create/list management; use hidden default HQ branch only; branch selector hidden when only one branch |
| `org.entity_kind_advanced` | Hide CONSOLIDATION/ELIMINATION; force OPERATING |
| `org.intercompany` | Hide IC partners/rules UI and routes |
| `org.business_unit` | Hide BU admin |
| `org.hierarchy_advanced` | Hide multi-hierarchy admin if sold separately |

### 9.5 Default path when packs are OFF

Single primary company + one implicit HQ branch → departments and other company-scoped features work. UI never requires the customer to “create a branch first” unless `org.multi_branch` is ON.

### 9.6 Implementation ownership

| Layer | Responsibility |
|-------|----------------|
| SaaS Admin / Platform | Store flags, billing, onboarding (company + default branch) |
| Organization BE | Optional guards on create second company/branch; always allow ops on primary + default branch |
| Organization FE | Hub cards, routes, filters, and forms respect flags; hide multi-entity noise |

**Now:** APIs remain open for development/demo. Enforcement + billing UI ships with platform packaging; this section is the binding product law for that work.

---

## 10. Change log

| Version | Summary |
|---------|---------|
| v1.0 | Roadmap + ADR-ORG-001 |
| **v1.1** | As-built DDL notes after P0–P7 implementation |
| v1.1+ | FE entity_kind copy; deferred feature-pack note (§9) |
| **v1.2** | §9 elevated to accepted product law: platform onboarding + default HQ branch + feature-gated org surfaces (2026-09-25) |
