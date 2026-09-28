# Organization — Sales & Purchasing Org Completion Plan v1.0

- **Document ID:** ORG-SALES-PURCH-PLAN-v1.0
- **Date:** 2026-09-28
- **Status:** Approved for execution (competitive parity without contradicting locked Org spine)
- **Owner Module:** Organization (Layer 5)
- **Related:** ORG_Layer_Completion_Roadmap_v1.0 (Package F), ORG_DDL_Notes_v1.1, ADR-ORG-001
- **Code:** akhlaghimhd/hamarehSaasErp · Front: akhlaghimhd/hamarehSaasErp-Front

---

## 1. Product stance

Bring Hamareh Sales/Purchasing organizational structures to **competitive parity** with SAP S/4HANA, NetSuite OneWorld, Dynamics 365, and Oracle Fusion **without breaking**:

1. Tenant → Company → Branch → Department operational spine
2. Business Unit as independent management dimension
3. Multi-purpose org hierarchies
4. RLS + Soft Delete + row_version + no cross-module physical FK
5. Feature-pack gating law (ORG_DDL_Notes §9)

**Boundary:** Organization owns structure only. Order-to-cash / procure-to-pay engines stay in future Sales/Purchasing modules.

---

## 2. Baseline (as-built 2026-09-28)

| Asset | Status |
|-------|--------|
| `erp_sales_organizations` | YES — code, name, company_id, is_active, RLS |
| `erp_purchasing_organizations` | YES — same |
| `erp_sales_org_assignments` / `erp_purch_org_assignments` | YES — company/branch |
| API list + create + assign | YES (minimal) |
| show / update / soft-delete / restore | NO |
| list assignments / unassign | NO |
| Distribution channel | NO |
| Product division | NO |
| Sales area (Org+Channel+Division) | NO |
| Sales office / sales group | NO |
| Reference purchasing org | NO |
| FE `/dashboard/organization/sales-purch` | NO (route may exist as stub) |
| PHPUnit coverage for full lifecycle | partial / missing |

---

## 3. Competitive feature matrix → decision

| Feature | SAP | NetSuite | D365 | Oracle | Hamareh decision |
|---------|-----|----------|------|--------|------------------|
| Sales Organization master | YES | via subsidiary/sales | via BU/territory | YES | **KEEP + enrich** |
| Purchasing Organization master | YES | weak | weak | YES | **KEEP + enrich** |
| Assign to company / plant | YES | subsidiary/location | legal entity / site | YES | **KEEP** |
| Distribution channel | YES | class/channel light | retail channel | light | **ADD** |
| Product division | YES | class | product hierarchy | light | **ADD** |
| Sales area (3-way combo) | YES | NO | NO | partial | **ADD** (SAP-class) |
| Sales office | YES | NO | territory-like | light | **ADD** |
| Sales group | YES | teams | teams | light | **ADD** |
| Reference / central purch org | YES | centralized subsidiary | centralized BU | YES | **ADD** `is_reference` |
| Territory management (CRM) | separate | territories | strong | strong | **DEFER** to CRM/Sales module |
| Cross-sub fulfillment | logistics | strong | strong | strong | **DEFER** to Inventory/Sales |
| Pricing procedures by sales area | SD | price levels | price groups | YES | **DEFER** to Sales pricing |

---

## 4. Prioritized work packages

### P0 — Complete existing masters (MUST, no schema break)

| ID | Work | Deliverable |
|----|------|-------------|
| SP-P0-01 | Sales Org full CRUD | show, update, soft-delete, restore, row_version++ |
| SP-P0-02 | Purch Org full CRUD | same |
| SP-P0-03 | Assignment list + soft-delete unassign | API + service |
| SP-P0-04 | Permissions seed consistency | organization.sales_org.* / purch_org.* |
| SP-P0-05 | PHPUnit lifecycle + isolation | SalesPurchOrgTest |

### P1 — SAP-class sales structure (MUST for competitive parity)

| ID | Work | Deliverable |
|----|------|-------------|
| SP-P1-01 | `erp_distribution_channels` | tenant-scoped catalog + RLS |
| SP-P1-02 | `erp_product_divisions` | tenant-scoped catalog + RLS |
| SP-P1-03 | `erp_sales_org_channels` | assign channel to sales org |
| SP-P1-04 | `erp_sales_org_divisions` | assign division to sales org |
| SP-P1-05 | `erp_sales_areas` | unique (tenant, sales_org, channel, division) active |
| SP-P1-06 | Services + API for channel/division/area | CRUD |
| SP-P1-07 | Tests for sales area uniqueness + isolation | PHPUnit |

### P2 — Sales office / group + reference purch (MUST)

| ID | Work | Deliverable |
|----|------|-------------|
| SP-P2-01 | `erp_sales_offices` + assign to sales areas | tables + API |
| SP-P2-02 | `erp_sales_groups` under office | tables + API |
| SP-P2-03 | `is_reference` on purchasing org | column + domain rule |
| SP-P2-04 | Optional `description` on sales/purch orgs | column |

### P3 — Frontend sales-purch admin (MUST after P0–P1 API stable)

| ID | Work | Deliverable |
|----|------|-------------|
| SP-P3-01 | Hub card + route `/dashboard/organization/sales-purch` | FE |
| SP-P3-02 | Tabs: Sales Orgs / Purch Orgs / Channels / Divisions / Sales Areas | FE |
| SP-P3-03 | Assignment UI company/branch | FE |
| SP-P3-04 | Feature-flag hook points (hide advanced when pack OFF) | FE ready |

### P4 — Hardening & docs

| ID | Work |
|----|------|
| SP-P4-01 | Update ORG_DDL_Notes with new tables |
| SP-P4-02 | PermissionSeeder + event names if needed |
| SP-P4-03 | Demo seeder sample sales area |
| SP-P4-04 | Regression suite green under app_user |

### Explicit non-goals (this plan)

- Territory engine / quota / commission
- Pricing condition tables
- Sales order / purchase order documents
- Cross-company fulfillment logistics
- Physical FK to Inventory / Accounting

---

## 5. Schema sketch (P1–P2)

All tables: `tenant_id`, soft delete, `row_version`, RLS FORCE + tenant_isolation_policy.

```
erp_distribution_channels (distribution_channel_id PK, code, name, is_active)
erp_product_divisions     (division_id PK, code, name, is_active)
erp_sales_org_channels    (id PK, sales_org_id, distribution_channel_id, is_active)
erp_sales_org_divisions   (id PK, sales_org_id, division_id, is_active)
erp_sales_areas           (sales_area_id PK, sales_org_id, distribution_channel_id, division_id, code?, name?, is_active)
                         UNIQUE (tenant_id, sales_org_id, distribution_channel_id, division_id) WHERE deleted_at IS NULL
erp_sales_offices         (sales_office_id PK, code, name, sales_org_id?, is_active)
erp_sales_office_areas    (id PK, sales_office_id, sales_area_id)
erp_sales_groups          (sales_group_id PK, sales_office_id, code, name, is_active)
erp_purchasing_organizations += is_reference boolean default false, description nullable
erp_sales_organizations     += description nullable
```

Logical refs only to `erp_companies` / `erp_branches` (already on assignments).

---

## 6. API surface (target)

Prefix: Organization module routes under `/api/.../organization`

- Sales org: GET/POST `/sales-organizations`, GET/PUT/DELETE `/sales-organizations/{id}`, POST restore
- Assignments: GET/POST `/sales-organizations/{id}/assignments`, DELETE assignment
- Same pattern for purchasing-organizations (+ `is_reference`)
- Channels: `/distribution-channels` CRUD
- Divisions: `/product-divisions` CRUD
- Sales areas: `/sales-areas` CRUD
- Sales offices / groups: `/sales-offices`, `/sales-groups`

Permissions family: `organization.sales_org.*`, `organization.purch_org.*`, `organization.sales_structure.view|manage`

---

## 7. Execution order

1. Docs (this file) on main  
2. SP-P0 backend + tests  
3. SP-P1 migrations + services + tests  
4. SP-P2  
5. SP-P3 frontend  
6. SP-P4 docs/seed/regression  

Commit messages English; user reports Persian. Docker: `docker compose exec app ...` only.

---

## 8. Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-09-28 | Initial competitive completion plan |
