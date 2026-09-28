# Organization — Sales & Purchasing Structure Status and Debt v1.0

- **Document ID:** ORG-SALES-PURCH-STATUS-v1.0
- **Date:** 2026-09-28
- **Related:** ORG_Sales_Purch_Completion_Plan_v1.0, ORG_DDL_Notes_v1.1
- **Code:** hamarehSaasErp @ 41596da5 · Front @ a7d8875f

---

## 1. As-built (closed for competitive structure parity)

| Area | Backend | Frontend | Tests |
|------|---------|----------|-------|
| Sales Organization CRUD + soft-delete | DONE | Tab list/create/delete | OrgP5 partial |
| Purchasing Organization + `is_reference` | DONE | Tab list/create/delete | OrgP5 partial |
| Distribution Channel | DONE | Tab | uniqueness test |
| Product Division | DONE | Tab | uniqueness test |
| Sales Area (Org+Channel+Division) | DONE | Tab | duplicate triple test |
| Sales Office | DONE | Tab «دفتر / گروه» | structure test |
| Sales Group under Office | DONE | Nested under office | structure test |
| Company/Branch assignments API | DONE | **Not in UI yet** | smoke |
| Permissions `sales_structure.*` | seeded | FE uses existing gates where present | — |

**Competitive verdict:** Organizational structure dimension is at SAP-class baseline for Sales Area + Office/Group. Remaining work is hardening and polish, not missing structural masters.

---

## 2. Hardening backlog (prioritized)

### H1 — High (correctness / UX of errors) — next sprint

| ID | Item | Notes |
|----|------|-------|
| H1-01 | Map service `Exception` messages to HTTP 422 with stable `code` | Persian messages already exist; controller should not return 500 |
| H1-02 | Form Requests for Sales Structure (channel/division/area/office/group) | Replace inline `$request->validate` for consistency |
| H1-03 | Unique code validation aligned with soft-delete | Unique among non-deleted rows only (partial unique index or service rule already) — document + enforce in DB where missing |
| H1-04 | FE: surface backend message on duplicate Sales Area / code | Already uses `ApiClientError.message`; verify 422 path |
| H1-05 | Assignment UI (company/branch) on Sales & Purch org detail or drawer | API exists; UI gap |

### H2 — Medium (lifecycle completeness)

| ID | Item | Notes |
|----|------|-------|
| H2-01 | Update + restore endpoints for Channel / Division / Area / Office / Group | Update exists for channel/division; area/office/group update/restore optional |
| H2-02 | Soft-delete cascade policy for groups when office deleted | Today independent soft-delete; define product rule |
| H2-03 | Prevent delete of channel/division if active Sales Area references it | Guard in service |
| H2-04 | PHPUnit: office/group uniqueness + office soft-delete | Extend OrgP5 |
| H2-05 | Demo seeder: one Sales Area + one Office/Group for Aria Sanat | Optional |

### H3 — Feature packs & hub (blocked on SaaS Admin catalog)

| ID | Item | Notes |
|----|------|-------|
| H3-01 | Feature flag for advanced sales structure (channel/division/area/office) | Per ORG_DDL_Notes §9; hide tabs when pack OFF |
| H3-02 | Hub card visibility gated by pack | organization-home |
| H3-03 | Document pack name in Platform Owner catalog | e.g. `org.sales_structure` independent of multi_company |

### H4 — Low / defer to Sales module

| ID | Item |
|----|------|
| H4-01 | Enforce Sales Area on sales documents |
| H4-02 | Pricing / customer master by Sales Area |
| H4-03 | Territory / quota / commission |
| H4-04 | Reference purch org contract inheritance rules |
| H4-05 | `erp_sales_org_channels` / `erp_sales_org_divisions` explicit link tables | Current model allows any channel/division in area; optional stricter SAP assignment |

---

## 3. Recommended next execution order

1. **H1-01 + H1-02** (422 + Form Requests)  
2. **H1-05** Assignment UI  
3. **H2-03** referential guards on delete  
4. **H2-04** tests  
5. **H3-*** when SaaS Admin feature catalog lands  

---

## 4. Explicit non-goals (unchanged)

Territory engine, pricing conditions, sales/purchase order documents, cross-company fulfillment, physical FK across modules.

---

## 5. Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-09-28 | Status after Office/Group FE + hardening backlog |
