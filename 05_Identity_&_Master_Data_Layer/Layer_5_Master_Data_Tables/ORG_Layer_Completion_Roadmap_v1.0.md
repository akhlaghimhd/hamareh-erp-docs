# Organization Layer Completion Roadmap v1.0

- **Document ID:** ORG-ROADMAP-v1.0
- **Version:** 1.0
- **Status:** Approved for execution (product direction: full enterprise capability)
- **Date:** 2026-09-23
- **Owner Module:** Organization (Layer 5 Master Data / Organizational Structure)
- **Code repo:** akhlaghimhd/hamarehSaasErp
- **Docs repo:** akhlaghimhd/hamareh-erp-docs
- **Execution order:** Backend first → Docs versioning with each phase → Frontend after backend acceptance
- **Priority policy:** All capability packages A–I are **MUST**. Sequencing is by dependency only, not by optional scope reduction.

---

## 1. Product stance (locked)

This roadmap treats Hamareh as a **full multi-entity enterprise ERP platform**, not an SMB-only skeleton.

- Every capability identified in the competitive gap analysis (SAP S/4HANA, Oracle Fusion, Dynamics 365 Finance, NetSuite OneWorld, Odoo) that we lack is in scope as **MUST**.
- Where industry models conflict, we adopt the **highest-capability design that does not destroy existing Tenant → Company → Branch → Department + RLS/Scope work**.
- Simplification for short-term convenience is **rejected** unless it would force a breaking rewrite of closed Layer 4 / RLS foundations.

---

## 2. Architecture decision record (six dilemmas) — highest viable tier

Related file: `ORG_Architecture_Decisions_ADR-ORG-001_v1.0.md`

| ID | Dilemma | Locked decision | Rationale vs existing code |
|----|---------|-----------------|----------------------------|
| ADR-ORG-01 | One Legal Entity table vs SAP Company + Company Code | **Single table `erp_companies` with `entity_kind`** (`OPERATING`, `CONSOLIDATION`, `ELIMINATION`) + parent tree | Keeps current PK/Scope/RLS; enables NetSuite/SAP-class consolidation roles without splitting Company Code |
| ADR-ORG-02 | Branch vs Plant | **Branch is the operational site; evolve to Plant-grade** via `branch_kind`, logistics flags, and **logical link to Inventory warehouses** | Preserves `erp_branches`; avoids parallel Plant table that would break current FKs |
| ADR-ORG-03 | Business Unit separate from Legal Entity | **YES — introduce `erp_business_units`** (management dimension; may span companies via assignment table) | Additive; does not remove Company/Branch |
| ADR-ORG-04 | Single tree vs multi-hierarchy with purpose | **YES — `erp_org_hierarchies` + `purpose` + nodes** (LEGAL, MANAGEMENT, TAX, ESTABLISHMENT, CUSTOM) | Additive reporting/policy layer; operational FK tree Company→Branch→Dept remains system of record for transactions |
| ADR-ORG-05 | Rich legal identity on Company vs only on Business Partner | **Both:** Company = internal Legal Entity SoT; BP = external party SoT | Aligns with existing BP docs; enriches `erp_companies` without merging domains |
| ADR-ORG-06 | Intercompany now vs placeholder | **Full intercompany + elimination is MUST**, implemented in dependency phases after GL-ready company attributes | Model fields and partner mapping land early; posting engine follows accounting maturity |

**Non-negotiable compatibility constraints**

1. Keep `tenant_id` isolation + PostgreSQL RLS patterns already delivered.
2. Keep Scope types `COMPANY` / `BRANCH` / `DEPARTMENT` (extend with `BUSINESS_UNIT` when BU ships).
3. No physical FK across module boundaries (logical UUID only to Inventory, Accounting, Identity users).
4. Soft delete + `row_version` + audit columns on all new operational tables.
5. Do not rename `company_id` / `branch_id` / `department_id` primary keys.

---

## 3. Work coding scheme

Format: `ORG-{PHASE}-{SEQ}-{SHORT}`

| Segment | Meaning |
|---------|---------|
| ORG | Organization layer track |
| PHASE | P0…P7 execution wave |
| SEQ | two-digit order inside phase |
| SHORT | stable mnemonic |

**Priority tag:** all items below are `MUST` unless explicitly marked `DOC` (documentation-only) or `FE` (frontend, deferred until backend phase accepted).

**Status values for tracking:** `TODO` | `IN_PROGRESS` | `DONE` | `BLOCKED`

---

## 4. Capability packages (all MUST)

### Package A — Legal entity master enrichment
| Code | Item | Backend deliverable |
|------|------|---------------------|
| ORG-P0-01-LEGAL-NAME | `legal_name` | column + model + DTO + API + tests |
| ORG-P0-02-TRADE-NAME | `trade_name` | same |
| ORG-P0-03-COMPANY-TYPE | `company_type` (enum/smallint per docs + extensions) | same |
| ORG-P0-04-TAX-IDS | `tax_identifier`, keep/map `economic_code`, optional `national_id`, `vat_registration` | same |
| ORG-P0-05-REG-META | `registration_date`, `registration_place`, `incorporation_country_id` (logical) | same |
| ORG-P0-06-STATUS | `status` smallint (Active/Suspended/Dissolving/Dissolved) + migrate from `is_active` | same |

### Package B — Group structure
| Code | Item | Backend deliverable |
|------|------|---------------------|
| ORG-P1-01-PRIMARY | `is_primary` + DB partial unique (one primary active per tenant) | migration + service rules + tests |
| ORG-P1-02-PARENT | `parent_company_id` (same-tenant tree) | migration + cycle guard + tests |
| ORG-P1-03-ENTITY-KIND | `entity_kind` OPERATING / CONSOLIDATION / ELIMINATION | migration + validation |
| ORG-P1-04-OWNERSHIP | `erp_company_ownerships` (percent, valid_from/to, relation_type) | table + CRUD service + tests |
| ORG-P1-05-ELIM-RULES | elimination subsidiary constraints (currency=parent, kind=ELIMINATION) | domain rules + tests |

### Package C — Financial attributes on legal entity
| Code | Item | Backend deliverable |
|------|------|---------------------|
| ORG-P2-01-CURRENCY | `base_currency_id` (logical UUID) | column + API |
| ORG-P2-02-FY-LINK | explicit company↔fiscal year contract (use/align existing fiscal tables) | service contract + tests |
| ORG-P2-03-COA-REF | `chart_of_accounts_id` logical ref | column + API |
| ORG-P2-04-FX-RATES | consolidation rate types support hooks (Current/Average/Historical) — structure for Accounting | interface + tables if owned by Org/Fin boundary decision |

### Package D — Address, contact, bank, officers
| Code | Item | Backend deliverable |
|------|------|---------------------|
| ORG-P0-07-ADDR | `entity_addresses` polymorphic (COMPANY, BRANCH, …) | migration + RLS + service + tests |
| ORG-P0-08-CONTACT | `entity_contact_points` polymorphic | same |
| ORG-P3-01-BANK | `erp_company_bank_accounts` | table + service + tests |
| ORG-P3-02-OFFICERS | `erp_company_officers` (role, person/user logical ref, mandate) | table + service + tests |

### Package E — Operational depth (Branch/Plant-grade, BU, Cost Center)
| Code | Item | Backend deliverable |
|------|------|---------------------|
| ORG-P3-03-BRANCH-KIND | `branch_kind` (OFFICE, PLANT, WAREHOUSE_SITE, DISTRIBUTION, MIXED) + logistics flags | migration + API |
| ORG-P3-04-BRANCH-WH | logical `default_warehouse_id` / site–warehouse mapping table | no physical FK to Inventory |
| ORG-P3-05-BRANCH-TREE | optional `parent_branch_id` multi-level branch | migration + cycle guard |
| ORG-P4-01-BU | `erp_business_units` + `erp_business_unit_companies` | tables + Scope type extension prep |
| ORG-P4-02-CC | `erp_cost_centers` per Master Data docs (company, optional dept, parent) | tables + service + tests |
| ORG-P4-03-DEPT-ALIGN | align `erp_departments` with docs (`company_id`, optional `branch_id`) without breaking existing rows | carefully migrated schema |

### Package F — Sales / Procurement org structures
| Code | Item | Backend deliverable |
|------|------|---------------------|
| ORG-P5-01-SALES-ORG | `erp_sales_organizations` (+ channels/divisions assignments as needed) | tables + API skeleton |
| ORG-P5-02-PURCH-ORG | `erp_purchasing_organizations` | tables + API skeleton |
| ORG-P5-03-ASSIGN | assignment tables to company/plant-branch | tables + validation |

### Package G — Multi-hierarchy & access dimensions
| Code | Item | Backend deliverable |
|------|------|---------------------|
| ORG-P4-04-HIER | `erp_org_hierarchies` (purpose, version, valid dates) | tables + service |
| ORG-P4-05-HIER-NODE | `erp_org_hierarchy_nodes` (parent/child, entity_type, entity_id) | tables + service |
| ORG-P6-01-CLASS | parallel dimensions `erp_org_classes` / location dimension if not covered by branch | tables |
| ORG-P6-02-SCOPE-BU | Scope type `BUSINESS_UNIT` + enforcement hooks | Identity/Org integration |
| ORG-P6-03-SHARE-POLICY | shared vs company-specific master data policy flags (platform-level convention) | ADR + config hooks |

### Package H — Intercompany & consolidation
| Code | Item | Backend deliverable |
|------|------|---------------------|
| ORG-P5-04-IC-MAP | intercompany partner mapping (company↔company as customer/vendor logical refs) | tables + service |
| ORG-P5-05-IC-RULE | intercompany transaction rules (doc type pairs) | tables + domain service |
| ORG-P6-04-ELIM-POST | elimination posting contract toward Accounting (events/API) | integration events v1 |
| ORG-P6-05-CONSOL | consolidation run placeholders (hierarchy snapshot, rate set ref) | structures + tests for Org side |

### Package I — Lifecycle / onboarding
| Code | Item | Backend deliverable |
|------|------|---------------------|
| ORG-P1-06-ONBOARD | primary company creation in real tenant onboarding pipeline (not only demo seeder) | service hook + tests |
| ORG-P6-06-ESC | enterprise structure configurator (API-driven template apply) | application service |

### Documentation tasks (versioned with phases)
| Code | Item |
|------|------|
| ORG-DOC-01 | This roadmap v1.0 (this file) |
| ORG-DOC-02 | ADR-ORG-001 architecture decisions |
| ORG-DOC-03 | Update `02_Master_Data_Table_Definitions.md` → v1.1 company/branch/BU/hierarchy DDL |
| ORG-DOC-04 | Permission catalog entries for new org resources |
| ORG-DOC-05 | Event catalog `organization.*.v1` additions |

### Frontend (after each backend wave acceptance) — tracked, not started in BE-only mode
| Code | Item |
|------|------|
| ORG-FE-P0 | Company form sections: legal, tax, status, address, contact |
| ORG-FE-P1 | Hierarchy tree, primary flag, ownership UI |
| ORG-FE-P2 | Currency / FY display bindings |
| ORG-FE-P3 | Bank, officers, branch-kind |
| ORG-FE-P4 | BU, cost center, hierarchy purpose browser |
| ORG-FE-P5 | Sales/Purch org admin (minimal) |
| ORG-FE-P6 | Intercompany mapping admin |

---

## 5. Execution waves (backend order)

### Wave P0 — Legal master + contact surface (foundation)
**Depends on:** nothing new  
**Codes:** ORG-P0-01 … ORG-P0-08, ORG-DOC-03 (partial)  
**Exit criteria:** migrations green; Company show/update API returns new fields; address/contact CRUD for `entity_type=COMPANY`; OrganizationCrudAndIsolationTest extended; RLS on new tables.

### Wave P1 — Group spine + onboarding
**Depends on:** P0  
**Codes:** ORG-P1-01 … ORG-P1-06  
**Exit criteria:** one primary company rule; parent tree without cycles; entity_kind validation; onboarding creates HQ operating company; ownership table CRUD.

### Wave P2 — Financial attributes
**Depends on:** P1  
**Codes:** ORG-P2-01 … ORG-P2-04  
**Exit criteria:** currency/CoA logical refs stored; FY contract documented and tested at boundary.

### Wave P3 — Bank, officers, plant-grade branch
**Depends on:** P0–P1  
**Codes:** ORG-P3-01 … ORG-P3-05  
**Exit criteria:** bank/officers APIs; branch_kind + optional parent_branch; warehouse logical mapping table.

### Wave P4 — BU, cost center, multi-hierarchy
**Depends on:** P1, P3  
**Codes:** ORG-P4-01 … ORG-P4-05  
**Exit criteria:** BU assignment; cost centers per docs; hierarchy purposes LEGAL/MANAGEMENT creatable via API.

### Wave P5 — Sales/Purch org + IC mapping
**Depends on:** P1, P3  
**Codes:** ORG-P5-01 … ORG-P5-05  
**Exit criteria:** org structures persisted; IC partner map + rules without requiring full posting engine.

### Wave P6 — Scope extension, share policy, consolidation hooks, ESC
**Depends on:** P4, P5, Accounting readiness for posting hooks  
**Codes:** ORG-P6-01 … ORG-P6-06, ORG-DOC-04, ORG-DOC-05  
**Exit criteria:** BUSINESS_UNIT scope wired; elimination/consolidation contracts published as events; configurator applies template.

### Wave P7 — Hardening
**Codes:** full regression, permission seeder, performance indexes, FE waves start  
**Exit criteria:** PHPUnit suite green under `app_user`; docs v1.1 published; FE can bind to stable APIs.

---

## 6. Suggested first backend commit slice (immediate next work)

Start **Wave P0** only after this document is on `main` of docs repo:

1. Migration: enrich `erp_companies` (A fields + nullable columns, backfill `legal_name` from `name`, `status` from `is_active`).
2. Migration: `entity_addresses`, `entity_contact_points` with tenant_id + RLS.
3. Model/Service/Request/Resource updates (full files).
4. Tests: isolation + validation.
5. Docs: DDL patch notes under ORG-DOC-03 draft section.

---

## 7. Traceability

| Source | Use |
|--------|-----|
| Competitive gap report (chat 2026-09-23) | Package A–I inventory |
| `02_Master_Data_Table_Definitions.md` | DDL baseline to version to 1.1 |
| Existing `erp_companies` / branches / departments migrations | Compatibility baseline |
| Layer 4 RLS/Scope | Non-break constraints |

---

## 8. Document control

| Version | Date | Author | Notes |
|---------|------|--------|-------|
| 1.0 | 2026-09-23 | Architecture (agent + product owner direction) | Full-enterprise MUST scope; BE-first |

**Citation name for future chats:** `ORG_Layer_Completion_Roadmap_v1.0`  
**Path:** `05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/ORG_Layer_Completion_Roadmap_v1.0.md`
