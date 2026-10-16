# Project Debt Register v1.0

**Document ID:** DEBT-REGISTER-v1.0  
**SSOT for open project debts across all layers/modules**  
**Created:** 2026-10-06  
**Last updated:** 2026-10-16T19:50:00+00:00  
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

**Addendum (2026-10-16 — user law):** Residual work is never “optional” without a debt row. Either implement and **CLOSED**, or keep **OPEN/DEFERRED** with concrete remaining work. Do not dismiss product-path items as non-blocking without registration.

---

## C. Cross-cutting / Platform blockers (index)

| Debt ID | Title | Status |
|---------|-------|--------|
| DEBT-PLT-001 | SaaS Admin feature catalog + pack purchase API | **CLOSED** |
| DEBT-PLT-002 | Dual-approval tenant settings UI | **CLOSED** |
| DEBT-PLT-003 | Tenant retention setting UI + purge job | **PARTIAL** |
| **DEBT-ORG-012** | Org hierarchy depth limits in Feature Pack / Platform Owner | **OPEN** |
| **DEBT-HOLD-W7** | Non-finance modules company operate (Inventory / Sales / Purch) | **OPEN** |

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
| **DEBT-ID-H01** | **CLOSED** — Wave 0 IDORG-T01–T04 delivered on develop |
| **DEBT-ID-H02** | **CLOSED** — Wave 1 IDORG-T05–T09 delivered on develop |
| **DEBT-ID-H03** | **CLOSED** — Wave 2 IDORG-T10–T14 delivered on develop |
| **DEBT-ID-H04** | **CLOSED** — Wave 3 IDORG-T15–T17 branch admin delivered on develop |

### DEBT-ORG-012 — Org hierarchy depth limits

| Field | Value |
|-------|--------|
| **1. Module** | Organization + SaasAdmin (Platform Owner) |
| **2. Section** | Max company / branch tree depth via Feature Pack |
| **3. Reason** | Unbounded depth is not desired for mid-size SaaS; product law requires pack-controlled ceiling (ADR-FIN-003 L16 + ADR-ID-ORG-004 G13 / IDORG-T19). |
| **4. Layer type** | Architecture + Backend + Platform Owner FE |
| **5. Created at** | 2026-10-14T06:40:00+02:00 |
| **6. Owner decision** | **OPEN — must implement.** Not deferred as optional. Blocked only on shipping FeatureCatalog keys + enforce in create services. |
| **7. Suggestion** | (1) FeatureCatalog keys e.g. `org.max_company_depth` / `org.max_branch_depth` (or pack tiers); (2) tenant entitlement; (3) `CompanyService::create` assert depth from primary root ≤ pack limit; (4) same for `BranchService`; (5) FA error on exceed; (6) Platform Owner admin UI for pack values. |
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
| **Status** | **CLOSED** (2026-10-15) — T01–T04 on develop; tests HoldingAccessServiceTest green |

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
| **Status** | **CLOSED** (2026-10-15) — T05–T09 delivered |

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
| **Status** | **CLOSED** (2026-10-15) — T10–T14 delivered |

### DEBT-ID-H04 — Branch as Identity admin unit

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore + Organization |
| **2. Section** | BRANCH scope on member admin |
| **3. Reason** | HoldingAccess is COMPANY-centric; branch manager product need open. |
| **4. Layer type** | Architecture + Backend + Frontend |
| **5. Created at** | 2026-10-14T07:30:00+02:00 |
| **6. Owner decision** | G09, G14 accepted — implement (user chose branch = Identity admin) |
| **7. Suggestion** | Wave 3 IDORG-T15–T17 |
| **Status** | **CLOSED** (2026-10-15) — T15–T17 branch admin on develop |

---

## F. Financial Accounting — ADR-FIN-002 / ADR-FIN-003

| Debt ID | Title | Status |
|---------|-------|--------|
| DEBT-FIN-CSN-001 | CoA per-company isolation | **CLOSED** |
| DEBT-FIN-CSN-002 | Sequence concurrency | **CLOSED** |
| DEBT-FIN-CSN-003 | FE report company UX | **CLOSED** |
| DEBT-FIN-HOLD-001 | Read vs operate split | **CLOSED** |
| DEBT-FIN-HOLD-002 | Group observer role + grant | **CLOSED** |
| DEBT-FIN-HOLD-003 | Branch document series from fiscal year | **CLOSED** |
| **DEBT-HOLD-W7** | Non-finance company operate isolation | **OPEN** |

### DEBT-FIN-CSN-001 — CoA per-company isolation

| Field | Value |
|-------|--------|
| **1. Module** | FinancialAccounting |
| **2. Section** | Chart of Accounts (ADR-FIN-003 L09) |
| **3. Reason** | Product locked CoA per company; was tenant-shared. |
| **4. Layer type** | Backend + Frontend |
| **5. Created at** | 2026-10-12 |
| **6. Owner decision** | FIN-HOLD-W4 |
| **7. Suggestion** | company_id on accounts + uniqueness + cloneTree |
| **Status** | **CLOSED** (2026-10-16) — BE complete + FE accounts-page company selector + operate gates + accountService/hooks companyId; tests CoaPerCompanyTest + JournalAndCoaFeatureTest green. |

### DEBT-FIN-HOLD-001 — Read vs operate split

| Field | Value |
|-------|--------|
| **1. Module** | FinancialAccounting |
| **2. Section** | FinanceCompanyScope + all FIN services + FE mutate pages |
| **3. Reason** | is_owner had group-wide write; ADR-FIN-003 L03 requires hard split. |
| **4. Layer type** | Backend + Frontend |
| **5. Created at** | 2026-10-14 |
| **6. Owner decision** | W1 BE + W2 FE |
| **7. Suggestion** | assertCanReadCompany vs assertCanOperateCompany |
| **Status** | **CLOSED** (2026-10-15) — W1 services + W2 FE gates |

### DEBT-FIN-HOLD-002 — Group observer role + grant

| Field | Value |
|-------|--------|
| **1. Module** | Identity + Finance |
| **2. Section** | identity.group.observe + parent grant |
| **3. Reason** | Explicit group-read product surface required by ADR-FIN-003 / ADR-ID-ORG-004. |
| **4. Layer type** | Backend + Frontend |
| **5. Created at** | 2026-10-14 |
| **6. Owner decision** | Shared with IDORG-T08/T09. **Grant surface for medium SaaS = existing Identity APIs/UI** (role assign + scope assign), not a separate subtree wizard. |
| **7. Suggestion** | Delivered path: (1) perm `identity.group.observe` + role template `group-observer`; (2) AccessLayerCatalog `finance.view` for observer / `finance.operate` for company-operator; (3) FE AssignRolesCard + AssignScopesCard gated by `canMutateMembers` (technical_owner / soft_sme only); (4) FE read-only banners on members + finance mutate pages for observer / technical owner. |
| **Status** | **CLOSED** (2026-10-16) — product grant path complete via Identity role/scope assignment; no separate wizard in scope. |

### DEBT-FIN-HOLD-003 — Branch document series from fiscal year

| Field | Value |
|-------|--------|
| **1. Module** | FinancialAccounting |
| **2. Section** | Document numbering L07 |
| **3. Reason** | Branch series from fiscal year start when enabled. |
| **4. Layer type** | Backend + Frontend |
| **5. Created at** | 2026-10-14 |
| **6. Owner decision** | FIN-HOLD-W3 |
| **7. Suggestion** | Company setting branch_series_from_fiscal_year |
| **Status** | **CLOSED** (2026-10-16) — BE+API+tests; FE company-numbering page; journals formBranchId + useFinanceCompanyScope wired on origin/develop. |

### DEBT-HOLD-W7 — Non-finance modules company operate isolation

| Field | Value |
|-------|--------|
| **1. Module** | Inventory (+ future Sales / Purch operational documents) |
| **2. Section** | FIN-HOLD-W7 / ADR-FIN-003 cross-module read≠operate |
| **3. Reason** | ADR requires the same holding isolation on non-finance modules. Inventory is currently **WAREHOUSE-scoped**, not company-operate. Sales/Purch commercial document engines are not shipped yet. |
| **4. Layer type** | Architecture + Backend (+ FE when module UIs exist) |
| **5. Created at** | 2026-10-16T19:50:00+00:00 |
| **6. Owner decision** | **OPEN — required by ADR.** Do not mark CLOSED until company-bound operational documents exist and use operate gates. Do not force FinanceCompanyScope onto pure warehouse masters. |
| **7. Suggestion** | When Inventory posts company-level documents: shared Holding operate assert (or FinanceCompanyScope-equivalent). When Sales/Purch ship: wire at document create/post. Track progress under this debt only. |
| **Status** | **OPEN** |

---

## G. Work breakdown SSOT

| Document | Purpose |
|----------|--------|
| `ID_ORG_Holding_Remediation_Work_Breakdown_v1.0.md` | Tasks IDORG-T01…T25 for gaps G01…G25 |
| `ADR-FIN-003` §4 | FIN-HOLD waves W0…W8 |

### Holding / FIN-HOLD wave status (2026-10-16)

| Wave | Deliverable | Status |
|------|-------------|--------|
| IDORG T01–T17 | Actor kinds, list/mutate, catalog, observer, branch admin, FE | **DONE** |
| FIN-HOLD W1 | assertCanRead vs assertCanOperate on FIN services | **DONE** |
| FIN-HOLD W2 | FE mutate gates | **DONE** |
| FIN-HOLD W3 | Branch series + FE numbering + journal branch_id | **DONE** |
| FIN-HOLD W4 | CoA per company + FE selector | **DONE** |
| FIN-HOLD W5 | AccessLayerCatalog + DefaultRoleCatalog finance.* | **DONE** |
| FIN-HOLD W6 | operate_denied + journal.deleted audit | **DONE** |
| FIN-HOLD W7 | Non-finance modules | **OPEN → DEBT-HOLD-W7** |
| Depth pack | DEBT-ORG-012 | **OPEN** |

---

## Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.5 | 2026-10-14 | DEBT-ORG-012 OPEN |
| 1.6 | 2026-10-14 | ADR-FIN-003; FIN-HOLD debts; CoA OPEN |
| 1.7 | 2026-10-14 | **ADR-ID-ORG-004** + work breakdown; DEBT-ID-H01…H04 OPEN |
| 1.8 | 2026-10-15 | H01–H04 CLOSED; FIN-HOLD-001 CLOSED; HOLD-002 PARTIAL; HOLD-003 OPEN |
| 1.9 | 2026-10-16 | FIN-HOLD-003 PARTIAL after W3 BE+API |
| 2.0 | 2026-10-16 | FIN-HOLD-W4 CoA BE PARTIAL (CSN-001); HOLD-003 residual journals push |
| 2.1 | 2026-10-16 | **CSN-001 CLOSED**; **HOLD-003 CLOSED** — FE residual path A complete |
| 2.2 | 2026-10-16 | **FIN-HOLD-W5** AccessLayerCatalog + DefaultRoleCatalog finance.*; **FIN-HOLD-W6** L15 audit CLOSED. Journal assertPostingAllowed order fixed. |
| **2.3** | **2026-10-16** | **HOLD-002 CLOSED** (grant = Identity role/scope assign). **DEBT-HOLD-W7 OPEN** registered (non-finance). **ORG-012** remains OPEN with concrete implement path. User law: no optional residual without debt row. |
