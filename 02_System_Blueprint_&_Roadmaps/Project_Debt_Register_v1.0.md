# Project Debt Register v1.0

**Document ID:** DEBT-REGISTER-v1.0  
**SSOT for open project debts across all layers/modules**  
**Created:** 2026-10-06  
**Last updated:** 2026-10-14T07:15:00+02:00  
**Repos:** akhlaghimhd/hamareh-erp-docs  

> **History note:** Full DEBT-ID-001…011 and DEBT-ORG-001…011 detail tables remain available at git commit `cf30dd8` (`git show cf30dd8:02_System_Blueprint_&_Roadmaps/Project_Debt_Register_v1.0.md`). This index carries current SAASADM close-out + live status for platform debts.

**Related:**
- `04_SaaS_Core_Platform_Layers/Layer_2_SaaS_Admin/SAASADM_BE_Closeout_P0-P5_2026-10-06.md`
- `04_SaaS_Core_Platform_Layers/Layer_2_SaaS_Admin/ADR-SAASADM-001_Feature_Pack_Model_v1.0.md`
- `06_Vertical_ERP_Modules/04_Financial_Accounting/ADR-FIN-002_Company_Scope_Access_and_Document_Numbering_v1.0.md`
- `06_Vertical_ERP_Modules/04_Financial_Accounting/ADR-FIN-002_Implementation_Status_v1.0.md`
- `06_Vertical_ERP_Modules/04_Financial_Accounting/ADR-FIN-003_Holding_Company_Access_and_Document_Isolation_Laws_v1.0.md`

---

## Register law (LOCKED 2026-10-06)

Every new debt entry **must** include: Module, Section, Reason, Layer type, Created at, Owner decision, Suggestion.  
Closed debts: mark **Status: CLOSED** (do not delete).  
Status values: OPEN | BLOCKED | PARTIAL | DEFERRED | CLOSED

---

## C. Cross-cutting / Platform blockers (index)

| Debt ID | Title | Status |
|---------|-------|--------|
| DEBT-PLT-001 | SaaS Admin feature catalog + pack purchase API | **CLOSED** — L1 FeatureCatalog + admin APIs + FE `/admin` shell |
| DEBT-PLT-002 | Dual-approval tenant settings UI | **CLOSED** — formal Identity tenant settings (`/dashboard/identity/settings`); dual-approval stays in `tenant_settings` |
| DEBT-PLT-003 | Tenant retention setting UI + purge job | **PARTIAL** — platform `retention.*` keys seeded; purge job Org masters P0 shipped; tenant override UX still OPEN |
| **DEBT-ORG-012** | **Org hierarchy depth limits in Feature Pack / Platform Owner** | **OPEN** — reminder: define max company/branch tree depth per pack when touching SaaS Admin feature packs |

---

## E. Layer 2 — SaaS Admin — FINAL (2026-10-06)

| Wave | Status |
|------|--------|
| P0–P5 Backend | **CLOSED** on `hamarehSaasErp` develop |
| P6 Notifications/Support | **DEFERRED** (skeleton exists) |
| P7 FE admin shell | **CLOSED** — `/admin` login, tenants, feature-packs, system-settings |

### DEBT-SAAS-001 — Frontend SaaS Admin shell

| Field | Value |
|-------|--------|
| **1. Module** | SaasAdmin (L2) + Front |
| **2. Section** | Admin shell, catalog, entitlement, system settings UI |
| **3. Reason** | Backend APIs live; needed separate `/admin` from tenant dashboard. |
| **4. Layer type** | Frontend |
| **5. Created at** | 2026-10-06T19:50:00+02:00 |
| **6. Owner decision** | Shipped: login, tenants list, feature-packs grant/revoke, system-settings. |
| **7. Suggestion** | Optional polish only (FA pack labels, richer tenant detail). |
| **Status** | **CLOSED** — 2026-10-06 |

### DEBT-SAAS-002 — Tenant dual-approval settings formalization

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore + Front |
| **2. Section** | Tenant identity settings |
| **3. Reason** | Dual-approval in `tenant_settings`; page was labeled temporary. |
| **4. Layer type** | Frontend |
| **5. Created at** | 2026-10-06T19:50:00+02:00 |
| **6. Owner decision** | `/dashboard/identity/settings` is formal SoT for tenant dual-approval; not platform system_settings. |
| **7. Suggestion** | Purge/retention job remains DEBT-ORG-007 / ID-003. |
| **Status** | **CLOSED** — 2026-10-06 |

### Org / Identity pack residuals (index)

| Debt ID | Status |
|---------|--------|
| DEBT-ORG-001 | PARTIAL — BE CLOSED; FE hub hints via enabled_codes |
| DEBT-ORG-002 | CLOSED (BE) |
| **DEBT-ORG-003** | **CLOSED** — 2026-10-06 |
| DEBT-ORG-008 | PARTIAL — BE CLOSED; hub shows pack hints |
| **DEBT-ORG-012** | **OPEN** — Org hierarchy max depth via Feature Pack / Platform Owner |
| DEBT-ID-009 | PARTIAL — catalog SoT exists; Holding H4/H5 product rules open |
| DEBT-ID-001…008, 010–011 | See full tables at `cf30dd8` |
| DEBT-ORG-004…011 | See full tables at `cf30dd8` |

### DEBT-ORG-003 — Sales/Purch structure feature-pack gating (detail)

| Field | Value |
|-------|--------|
| **1. Module** | Organization (L5) + SaasPlatform feature catalog |
| **2. Section** | Sales/Purch structure masters create paths (H3) |
| **3. Reason** | Competitive sales/purch masters existed without sellable pack gates; L2 catalog was the prerequisite. |
| **4. Layer type** | Backend + Frontend |
| **5. Created at** | 2026-09-29 (track debt); formal close-out 2026-10-06 |
| **6. Owner decision** | Independent packs `org.sales_structure` and `org.purch_structure`. Create blocked without entitlement (403). List/read remain available (freeze semantics). FE hub shows packHint; no full list rewrite. |
| **7. Suggestion** | Optional: mirror same one-line assert into `SalesStructureService::create*` for defense-in-depth if callers bypass controller. |
| **Status** | **CLOSED** — 2026-10-06 |

### DEBT-ORG-012 — Org hierarchy depth limits (Feature Pack / Platform Owner)

| Field | Value |
|-------|--------|
| **1. Module** | Organization (L5) + SaasAdmin (L2) Feature Pack catalog |
| **2. Section** | Platform Owner entitlement — max depth of company tree / branch tree a tenant may create |
| **3. Reason** | Org model supports multi-level company and nested branch; mid-size SaaS customers need sellable depth caps. |
| **4. Layer type** | Architecture + Backend + Organization create guards |
| **5. Created at** | 2026-10-14T06:40:00+02:00 |
| **6. Owner decision** | OPEN until Feature Pack catalog refinement; see ADR-FIN-003 L16. |
| **7. Suggestion** | Pack keys for max company/branch depth; gate CompanyService/BranchService create. |
| **Status** | **OPEN** |

---

## F. Financial Accounting — ADR-FIN-002 / ADR-FIN-003

| Debt ID | Title | Status |
|---------|-------|--------|
| DEBT-FIN-CSN-001 | CoA per-company isolation | **OPEN** — product locked per-company in ADR-FIN-003 L09 (was DEFERRED tenant-shared) |
| DEBT-FIN-CSN-002 | Multi-process sequence race beyond lockForUpdate | **CLOSED** |
| DEBT-FIN-CSN-003 | FE company selector consistency on report surfaces | **CLOSED** |
| **DEBT-FIN-HOLD-001** | Split group **read** vs company **operate** write | **OPEN** — ADR-FIN-003 L01–L03, L06; `is_owner` must not auto-write all companies |
| **DEBT-FIN-HOLD-002** | Group observer role + parent grant of read scope | **OPEN** — ADR-FIN-003 L02, L05, L06 |
| **DEBT-FIN-HOLD-003** | Optional branch document series from fiscal year start | **OPEN** — ADR-FIN-003 L07, L10 |

### DEBT-FIN-CSN-001 — CoA per-company (detail)

| Field | Value |
|-------|--------|
| **1. Module** | FinancialAccounting |
| **2. Section** | Chart of Accounts |
| **3. Reason** | ADR-FIN-003 L09 locks CoA per company; code still tenant-shared. |
| **4. Layer type** | Architecture / Backend / Frontend |
| **5. Created at** | 2026-10-12T18:00:00+02:00 |
| **6. Owner decision** | **Per company** (2026-10-14). Implement migration + uniqueness per company. |
| **7. Suggestion** | Migration strategy: assign existing tree to primary company then clone for siblings, or explicit company_id + backfill. |
| **Status** | **OPEN** |

### DEBT-FIN-HOLD-001 — Read vs operate split

| Field | Value |
|-------|--------|
| **1. Module** | FinancialAccounting + IdentityCore |
| **2. Section** | FinanceCompanyScope / all FIN write services |
| **3. Reason** | Owner locked: holding may view/filter subsidiary data but must not post/edit child books; current is_owner group-wide write violates this. |
| **4. Layer type** | Backend (then FE) |
| **5. Created at** | 2026-10-14T07:15:00+02:00 |
| **6. Owner decision** | Implement assertCanReadCompany vs assertCanOperateCompany; fix journals and all FIN modules. |
| **7. Suggestion** | Wave W1 in ADR-FIN-003 §4. |
| **Status** | **OPEN** |

### DEBT-FIN-HOLD-002 — Group observer role

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore + Front |
| **2. Section** | Permissions / roles / grant UI |
| **3. Reason** | L02 requires explicit group read capability granted by parent, not flat owner power. |
| **4. Layer type** | Backend + Frontend |
| **5. Created at** | 2026-10-14T07:15:00+02:00 |
| **6. Owner decision** | New permission(s) + role surface; parent assigns observe scope. |
| **7. Suggestion** | Wave W0 in ADR-FIN-003 §4. |
| **Status** | **OPEN** |

### DEBT-FIN-HOLD-003 — Branch series optional from fiscal year

| Field | Value |
|-------|--------|
| **1. Module** | FinancialAccounting |
| **2. Section** | Document sequences + company finance settings |
| **3. Reason** | L07: default company series; optional branch series from start of fiscal year only; company-level docs without branch keep company series. |
| **4. Layer type** | Backend + Frontend |
| **5. Created at** | 2026-10-14T07:15:00+02:00 |
| **6. Owner decision** | Setting per company; no mid-period switch. |
| **7. Suggestion** | Wave W3 in ADR-FIN-003 §4. |
| **Status** | **OPEN** |

### DEBT-FIN-CSN-002 — Sequence concurrency (detail)

| Field | Value |
|-------|--------|
| **1. Module** | FinancialAccounting |
| **2. Section** | `fin_document_sequences` / JournalEntryService |
| **3. Reason** | F3-01 concurrency proof. |
| **4. Layer type** | Backend |
| **5. Created at** | 2026-10-12T18:00:00+02:00 |
| **6. Owner decision** | lockForUpdate + sequential test accepted. |
| **7. Suggestion** | Optional load test outside PHPUnit. |
| **Status** | **CLOSED** — 2026-10-12 |

### DEBT-FIN-CSN-003 — FE report company UX (detail)

| Field | Value |
|-------|--------|
| **1. Module** | FinancialAccounting Front |
| **2. Section** | reports / tax-reports / period-close |
| **3. Reason** | Consistent company selector. |
| **4. Layer type** | Frontend |
| **5. Created at** | 2026-10-12T18:00:00+02:00 |
| **6. Owner decision** | useFinanceCompanyScope shipped on key report pages. |
| **7. Suggestion** | Align with operate vs read scope when HOLD-001 lands. |
| **Status** | **CLOSED** — 2026-10-12 |

---

## Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-10-06 | Initial register |
| 1.1 | 2026-10-06 | SAASADM-P0/P1 BE verified |
| 1.2 | 2026-10-06 | SAASADM-P2–P7 closed |
| 1.3 | 2026-10-06 | DEBT-ORG-003 CLOSED |
| 1.4 | 2026-10-12 | DEBT-FIN-CSN close-out |
| 1.5 | 2026-10-14 | DEBT-ORG-012 OPEN |
| 1.6 | 2026-10-14 | **ADR-FIN-003 locked**; DEBT-FIN-HOLD-001/002/003 OPEN; DEBT-FIN-CSN-001 → OPEN (per-company CoA) |
