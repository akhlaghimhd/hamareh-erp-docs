# Project Debt Register v1.0

**Document ID:** DEBT-REGISTER-v1.0  
**SSOT for open project debts across all layers/modules**  
**Created:** 2026-10-06  
**Last updated:** 2026-10-12T20:15:00+02:00  
**Repos:** akhlaghimhd/hamareh-erp-docs  

> **History note:** Full DEBT-ID-001…011 and DEBT-ORG-001…011 detail tables remain available at git commit `cf30dd8` (`git show cf30dd8:02_System_Blueprint_&_Roadmaps/Project_Debt_Register_v1.0.md`). This index carries current SAASADM close-out + live status for platform debts.

**Related:**
- `04_SaaS_Core_Platform_Layers/Layer_2_SaaS_Admin/SAASADM_BE_Closeout_P0-P5_2026-10-06.md`
- `04_SaaS_Core_Platform_Layers/Layer_2_SaaS_Admin/ADR-SAASADM-001_Feature_Pack_Model_v1.0.md`
- `06_Vertical_ERP_Modules/04_Financial_Accounting/ADR-FIN-002_Company_Scope_Access_and_Document_Numbering_v1.0.md`
- `06_Vertical_ERP_Modules/04_Financial_Accounting/ADR-FIN-002_Implementation_Status_v1.0.md`

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
| **DEBT-ORG-003** | **CLOSED** — 2026-10-06: packs `org.sales_structure` / `org.purch_structure`; create gated on SalesOrg/PurchOrg services + SalesStructureController store*; FE hub packHint; tests `SalesPurchFeaturePackGateTest` |
| DEBT-ORG-008 | PARTIAL — BE CLOSED; hub shows pack hints |
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

---

## F. Financial Accounting — ADR-FIN-002 (FIN-CSN) residuals

| Debt ID | Title | Status |
|---------|-------|--------|
| DEBT-FIN-CSN-001 | CoA per-company isolation | **DEFERRED** — product keeps tenant-level CoA |
| DEBT-FIN-CSN-002 | Multi-process sequence race beyond lockForUpdate | **CLOSED** — sequential test + row lock; OS parallel out of PHPUnit scope |
| DEBT-FIN-CSN-003 | FE company selector consistency on report surfaces | **CLOSED** — `useFinanceCompanyScope` + reports / tax / period-close |

### DEBT-FIN-CSN-001 — CoA per-company (detail)

| Field | Value |
|-------|--------|
| **1. Module** | FinancialAccounting |
| **2. Section** | Chart of Accounts (F2-02) |
| **3. Reason** | ADR matrix listed CoA access ∩ company policy; current CoA has no `company_id` (tenant shared tree). |
| **4. Layer type** | Architecture / Backend |
| **5. Created at** | 2026-10-12T18:00:00+02:00 |
| **6. Owner decision** | Keep tenant-level CoA until explicit product ADR change. Scope on journals/reports is sufficient for legal-entity isolation. |
| **7. Suggestion** | If per-company CoA is required later: new ADR amendment + migration `company_id` + uniqueness per company. |
| **Status** | **DEFERRED** |

### DEBT-FIN-CSN-002 — Sequence concurrency (detail)

| Field | Value |
|-------|--------|
| **1. Module** | FinancialAccounting |
| **2. Section** | `fin_document_sequences` / JournalEntryService |
| **3. Reason** | F3-01 required concurrency proof; PHPUnit cannot safely simulate true multi-process. |
| **4. Layer type** | Backend |
| **5. Created at** | 2026-10-12T18:00:00+02:00 |
| **6. Owner decision** | `lockForUpdate` inside transaction + `DocumentSequenceConcurrencyTest` sequential uniqueness accepted as ADR close criterion. |
| **7. Suggestion** | Optional load test outside PHPUnit if production contention appears. |
| **Status** | **CLOSED** — 2026-10-12 |

### DEBT-FIN-CSN-003 — FE report company UX (detail)

| Field | Value |
|-------|--------|
| **1. Module** | FinancialAccounting Front |
| **2. Section** | reports / tax-reports / period-close company filter |
| **3. Reason** | ADR L01–L02 required consistent company selector; some pages only defaulted primary without shared helper. |
| **4. Layer type** | Frontend |
| **5. Created at** | 2026-10-12T18:00:00+02:00 |
| **6. Owner decision** | Shared `useFinanceCompanyScope`; multi → selector; single → fixed label; wired on reports, tax-reports, period-close. Other FIN pages may adopt same hook when touched. |
| **7. Suggestion** | Apply hook to treasury/open-items/fixed-assets when next UX pass. |
| **Status** | **CLOSED** — 2026-10-12 |

---

## Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-10-06 | Initial register |
| 1.1 | 2026-10-06 | SAASADM-P0/P1 BE verified |
| 1.2 | 2026-10-06 | SAASADM-P2–P7 closed; DEBT-SAAS-001/002 CLOSED; PLT-001/002 CLOSED; PLT-003 PARTIAL |
| 1.3 | 2026-10-06 | DEBT-ORG-003 CLOSED (Sales/Purch pack gates + FE hub hint) |
| 1.4 | 2026-10-12 | DEBT-FIN-CSN-001 DEFERRED; 002/003 CLOSED; ADR-FIN-002 implementation close-out |
