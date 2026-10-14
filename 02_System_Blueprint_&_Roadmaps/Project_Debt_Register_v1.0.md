# Project Debt Register v1.0

**Document ID:** DEBT-REGISTER-v1.0  
**SSOT for open project debts across all layers/modules**  
**Created:** 2026-10-06  
**Last updated:** 2026-10-14T07:30:00+02:00  
**Repos:** akhlaghimhd/hamareh-erp-docs  

> **History note:** Full DEBT-ID-001…011 and DEBT-ORG-001…011 detail tables remain available at git commit `cf30dd8`.

**Related:**
- `04_SaaS_Core_Platform_Layers/Layer_2_SaaS_Admin/ADR-SAASADM-001_Feature_Pack_Model_v1.0.md`
- `05_Identity_&_Master_Data_Layer/ADR-ID-ORG-003_Holding_Access_Model_and_Delegated_Admin_v1.0.md`
- `05_Identity_&_Master_Data_Layer/ADR-ID-ORG-004_Holding_Identity_Org_Gaps_and_Remediation_v1.0.md`
- `05_Identity_&_Master_Data_Layer/ID_ORG_Holding_Remediation_Work_Breakdown_v1.0.md`
- `06_Vertical_ERP_Modules/04_Financial_Accounting/ADR-FIN-002_Company_Scope_Access_and_Document_Numbering_v1.0.md`
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
| DEBT-PLT-001 | SaaS Admin feature catalog + pack purchase API | **CLOSED** |
| DEBT-PLT-002 | Dual-approval tenant settings UI | **CLOSED** |
| DEBT-PLT-003 | Tenant retention setting UI + purge job | **PARTIAL** |
| **DEBT-ORG-012** | Org hierarchy depth limits in Feature Pack / Platform Owner | **OPEN** |

---

## E. Layer 2 — SaaS Admin / Org pack residuals (index)

| Debt ID | Status |
|---------|--------|
| DEBT-ORG-001 | PARTIAL |
| DEBT-ORG-002 | CLOSED |
| DEBT-ORG-003 | CLOSED |
| DEBT-ORG-008 | PARTIAL |
| **DEBT-ORG-012** | **OPEN** — depth caps via pack |
| DEBT-ID-009 | PARTIAL |
| **DEBT-ID-H01** | **OPEN** — Holding Identity power boundary (ADR-ID-ORG-004 G01–G04) |
| **DEBT-ID-H02** | **OPEN** — Group observer + role catalog ceilings (G05, G15–G17) |
| **DEBT-ID-H03** | **OPEN** — Child company admin completeness (G06–G08, G10, G25) |
| **DEBT-ID-H04** | **OPEN** — Branch as Identity admin unit (G09, G14) |

### DEBT-ORG-012 — Org hierarchy depth limits

| Field | Value |
|-------|--------|
| **1. Module** | Organization + SaasAdmin Feature Pack |
| **2. Section** | Max company/branch tree depth |
| **3. Reason** | Unbounded depth not desired for mid-size SaaS; pack must limit. |
| **4. Layer type** | Architecture + Backend |
| **5. Created at** | 2026-10-14T06:40:00+02:00 |
| **6. Owner decision** | OPEN; ADR-FIN-003 L16 + ADR-ID-ORG-004 G13 |
| **7. Suggestion** | See IDORG-T19 work breakdown |
| **Status** | **OPEN** |

### DEBT-ID-H01 — Holding Identity power boundary

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore |
| **2. Section** | is_owner / empty COMPANY scope / role CRUD power |
| **3. Reason** | Owner and shared role catalog allow permission inflation; conflicts with holding-observe model. |
| **4. Layer type** | Backend + Architecture |
| **5. Created at** | 2026-10-14T07:30:00+02:00 |
| **6. Owner decision** | Accepted remediation G01–G04 in ADR-ID-ORG-004 |
| **7. Suggestion** | Wave 0 tasks IDORG-T01–T04 |
| **Status** | **OPEN** |

### DEBT-ID-H02 — Group observer and catalog shape

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore + Front |
| **2. Section** | Observer role, seeds, permission tags |
| **3. Reason** | No first-class group observer; demo seeds likely flat-full. |
| **4. Layer type** | Backend + Frontend + Seeders |
| **5. Created at** | 2026-10-14T07:30:00+02:00 |
| **6. Owner decision** | G05, G15–G17 accepted |
| **7. Suggestion** | Wave 1 IDORG-T05–T09 (shared with FIN-HOLD-W0) |
| **Status** | **OPEN** |

### DEBT-ID-H03 — Child company delegated admin completeness

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore + Front |
| **2. Section** | Members, role assign, scope grant |
| **3. Reason** | COMPANY scope path exists but role CRUD/picker and FE still incomplete vs product. |
| **4. Layer type** | Backend + Frontend |
| **5. Created at** | 2026-10-14T07:30:00+02:00 |
| **6. Owner decision** | G06–G08, G10, G25 accepted |
| **7. Suggestion** | Wave 2 IDORG-T10–T14 |
| **Status** | **OPEN** |

### DEBT-ID-H04 — Branch as Identity admin unit

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore + Organization |
| **2. Section** | BRANCH scope on member admin |
| **3. Reason** | HoldingAccess is COMPANY-centric; branch manager product need open. |
| **4. Layer type** | Architecture + Backend + Frontend |
| **5. Created at** | 2026-10-14T07:30:00+02:00 |
| **6. Owner decision** | G09, G14 accepted — implement or explicit non-goal after Wave 3 decision |
| **7. Suggestion** | Wave 3 IDORG-T15–T17 |
| **Status** | **OPEN** |

---

## F. Financial Accounting — ADR-FIN-002 / ADR-FIN-003

| Debt ID | Title | Status |
|---------|-------|--------|
| DEBT-FIN-CSN-001 | CoA per-company isolation | **OPEN** |
| DEBT-FIN-CSN-002 | Sequence concurrency | **CLOSED** |
| DEBT-FIN-CSN-003 | FE report company UX | **CLOSED** |
| DEBT-FIN-HOLD-001 | Read vs operate split | **OPEN** |
| DEBT-FIN-HOLD-002 | Group observer role + grant | **OPEN** |
| DEBT-FIN-HOLD-003 | Branch document series from fiscal year | **OPEN** |

Detail tables for FIN debts: see register history / prior sections in git; status above is authoritative index.

---

## G. Work breakdown SSOT

| Document | Purpose |
|----------|--------|
| `ID_ORG_Holding_Remediation_Work_Breakdown_v1.0.md` | Tasks IDORG-T01…T25 for gaps G01…G25 |
| `ADR-FIN-003` §4 | FIN-HOLD waves W0…W8 |

---

## Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.5 | 2026-10-14 | DEBT-ORG-012 OPEN |
| 1.6 | 2026-10-14 | ADR-FIN-003; FIN-HOLD debts; CoA OPEN |
| 1.7 | 2026-10-14 | **ADR-ID-ORG-004** + work breakdown; DEBT-ID-H01…H04 OPEN |
