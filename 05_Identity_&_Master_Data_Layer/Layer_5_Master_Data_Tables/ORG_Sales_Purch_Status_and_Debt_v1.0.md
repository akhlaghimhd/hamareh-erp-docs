# Organization — Sales & Purchasing Structure Status and Debt v1.1

- **Document ID:** ORG-SALES-PURCH-STATUS-v1.1
- **Date:** 2026-09-29
- **Related:** ORG_Sales_Purch_Completion_Plan_v1.0, ORG_DDL_Notes_v1.1
- **Code:** hamarehSaasErp (H1/H2 closed) · Front (assignment UI + table UX closed)

---

## 1. As-built (closed for competitive structure parity + hardening)

| Area | Backend | Frontend | Tests |
|------|---------|----------|-------|
| Sales Organization CRUD + soft-delete | DONE | Tab list/create/delete + assignment_count | OrgP5 |
| Purchasing Organization + `is_reference` | DONE | Tab list/create/delete + assignment_count | OrgP5 |
| Distribution Channel | DONE | Tab | uniqueness test |
| Product Division | DONE | Tab | uniqueness test |
| Sales Area (Org+Channel+Division) | DONE | Tab | duplicate triple test |
| Sales Office | DONE | Tab «دفتر / گروه» | structure test |
| Sales Group under Office | DONE | Nested under office | structure test |
| Company/Branch assignments API + delete guards | DONE | Detail pages sales/[id] + purch/[id] | smoke + guards |
| DomainException → HTTP 422 | DONE (global handler) | ApiClientError surfaces message | — |
| Permissions `sales_structure.*` | seeded | FE uses existing gates where present | — |

**Competitive verdict:** Organizational structure dimension is at SAP-class baseline for Sales Area + Office/Group. H1 + H2 hardening closed 2026-09-29. Remaining open work is **feature-pack gating (H3)** and **Sales-module consumers (H4)** — not missing structural masters.

---

## 2. Closed hardening (do not re-open)

### H1 — High (correctness / UX) — **CLOSED 2026-09-29**

| ID | Item | Status |
|----|------|--------|
| H1-01 | Map service DomainException → HTTP 422 | DONE |
| H1-02 | Form Requests / consistent validation path | DONE (service + 422 path) |
| H1-03 | Unique code among non-deleted rows | DONE |
| H1-04 | FE surfaces backend 422 message | DONE |
| H1-05 | Assignment UI (company/branch) on Sales & Purch org detail | DONE (detail routes + CRUD) |

### H2 — Medium (lifecycle) — **CLOSED 2026-09-29**

| ID | Item | Status |
|----|------|--------|
| H2-01 | Update + restore for structure entities | DONE where required |
| H2-02 | Soft-delete cascade office → groups | DONE |
| H2-03 | Prevent delete when active Sales Area / assignment references | DONE (service guards + FE disable) |
| H2-04 | PHPUnit uniqueness + delete guards | DONE |
| H2-05 | DemoSalesStructureSeeder (H2-05) | DONE |

**Note:** `AriaSanatDemoOrgSeeder` was temporarily stubbed after a push incident; restore full body from commit `a206128c` only if a rich multi-company demo org is needed again. Structure demo path uses `DemoSalesStructureSeeder`.

---

## 3. Open debt (parked — resume after prerequisites)

### H3 — Feature packs & hub (**BLOCKED on SaaS Admin / Platform Owner catalog**)

| ID | Item | Notes |
|----|------|-------|
| H3-01 | Feature flag for advanced sales structure (channel/division/area/office) | Per ORG_DDL_Notes §9; hide advanced tabs when pack OFF; keep Sales/Purch Org always available on default path |
| H3-02 | Hub card visibility gated by pack | organization-home |
| H3-03 | Document pack name in Platform Owner catalog | e.g. `org.sales_structure` **independent** of `multi_company` / `multi_branch` |

**Prerequisite:** SaaS Admin feature catalog + per-tenant purchased packs API.

**Do not** invent a second feature-flag table inside Organization.

### H4 — Defer to Sales / Purchasing modules (out of Org layer)

| ID | Item | Owner module when ready |
|----|------|-------------------------|
| H4-01 | Enforce Sales Area on sales documents | Sales |
| H4-02 | Pricing / customer master by Sales Area | Sales / Pricing |
| H4-03 | Territory / quota / commission | Sales |
| H4-04 | Reference purch org contract inheritance rules | Purchasing |
| H4-05 | Optional `erp_sales_org_channels` / `erp_sales_org_divisions` explicit link tables | Org only if product requires stricter SAP-style assignment after H3 |

---

## 4. Recommended resume order

1. **H3-*** when SaaS Admin feature catalog lands (same wave as Smart Hierarchy D1/D7).
2. **H4-*** only after Sales/Purch document engines exist — bind to existing masters; do not reinvent org tables.

---

## 5. Explicit non-goals (unchanged)

Territory engine, pricing conditions, sales/purchase order documents, cross-company fulfillment, physical FK across modules.

---

## 6. Cross-references (other Org debts still open)

| Debt source | Open items | Blocker |
|-------------|------------|---------|
| ORG_Smart_Hierarchy_Status_and_Debt_v1.0 | **D1** (pack upgrade/downgrade), **D7** (CUSTOM hierarchy pack) | SaaS Admin catalog |
| ORG_Smart_Hierarchy_Status_and_Debt_v1.0 | D5 large-tenant queue tuning | Ops / scale |
| ORG_Intercompany_Status_and_Debt_v1.0 | Acc-IC-P1, Ops-IC-P1, IC-D-PLT-01 | Accounting + Sales/Purch + Platform pack enforce |

Identity debts (GET user-roles, change-password) are **closed** and must not be re-opened as Org work.

---

## 7. Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-09-28 | Status after Office/Group FE + hardening backlog |
| 1.1 | 2026-09-29 | H1+H2 closed; H3/H4 parked; assignment UI + 422 + delete guards as-built |
