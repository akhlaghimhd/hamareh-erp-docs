# Project Debt Register v1.0

**Document ID:** DEBT-REGISTER-v1.0  
**SSOT for open project debts across Identity / Org / Finance / Platform**  
**Version:** 2.5  
**Last updated:** 2026-10-18

> Closed debts are retained with Status: CLOSED (never deleted).

---

## How to register a debt (user law 2026-10-06)

Each debt row MUST include:
1. Module
2. Module section
3. Reason the debt was created
4. Layer split (BE / FE / Architecture)
5. Created-at timestamp
6. Owner decision
7. Suggested remediation (optional)

---

## A. Identity residuals (index)

| Debt ID | Title | Status |
|---------|-------|--------|
| DEBT-ID-009 | Soft-delete retention UX for members/roles/scopes | **PARTIAL** |
| DEBT-ID-H01 | Holding Identity power boundary | **CLOSED** |
| DEBT-ID-H02 | Soft SME vs multi-company empty scopes | **CLOSED** |
| DEBT-ID-H03 | Identity list vs mutate split | **CLOSED** |
| DEBT-ID-H04 | Role catalog gate | **CLOSED** |

---

## B. Organization residuals (index)

| Debt ID | Title | Status |
|---------|-------|--------|
| DEBT-ORG-001 | Feature-pack gating residual surfaces | **PARTIAL** |
| DEBT-ORG-002 | Sales/Purch structure H3 pack gate (legacy note) | **CLOSED** |
| DEBT-ORG-003 | Sales/Purch feature packs BE+FE | **CLOSED** |
| DEBT-ORG-008 | Rich company-detail panels restore | **PARTIAL** |
| DEBT-ORG-012 | Org hierarchy depth limits in Feature Pack / Platform Owner | **OPEN** |

---

## C. Cross-cutting / Platform blockers (index)

| Debt ID | Title | Status |
|---------|-------|--------|
| DEBT-PLT-001 | SaaS Admin feature catalog + pack purchase API | **CLOSED** |
| DEBT-PLT-002 | Dual-approval tenant settings UI | **CLOSED** |
| DEBT-PLT-003 | Tenant retention setting UI + purge job | **PARTIAL** |
| **DEBT-ORG-012** | Org hierarchy depth limits in Feature Pack / Platform Owner | **OPEN** |
| **DEBT-HOLD-W7** | Non-finance modules company operate (Inventory / Sales / Purch) | **OPEN** |
| **DEBT-ID-CEILING-001** | Membership anchor & hierarchical scope ceiling (ADR-ID-ORG-005) | **CLOSED** |

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
| **DEBT-ID-H04** | **CLOSED** — role catalog gate |

---

### DEBT-ORG-012 — Org hierarchy depth limits

| Field | Value |
|-------|--------|
| **1. Module** | Organization (+ Platform Owner feature packs) |
| **2. Section** | Company / Branch tree depth |
| **3. Reason** | Unlimited parent depth can create pathological trees; product wants pack-gated max depth. |
| **4. Layer type** | Architecture + Backend + Platform Admin |
| **5. Created at** | 2026-10-14 |
| **6. Owner decision** | Enforce via Feature Pack keys; CompanyService/BranchService assert |
| **7. Suggestion** | Concrete pack keys + service enforce + tests |
| **Status** | **OPEN** |

---

### DEBT-HOLD-W7 — Non-finance modules company operate isolation

| Field | Value |
|-------|--------|
| **1. Module** | Inventory / Sales / Purchasing (cross-module) |
| **2. Section** | FIN-HOLD-W7 / ADR-FIN-003 cross-module read≠operate |
| **3. Reason** | Finance read≠operate delivered; other modules not shipped with same split. |
| **4. Layer type** | Backend + FE when modules ship |
| **5. Created at** | 2026-10-16 |
| **6. Owner decision** | Park until Inventory/Sales/Purch operate surfaces exist |
| **7. Suggestion** | Mirror FinanceCompanyScope assertCanOperate pattern per module |
| **Status** | **OPEN** |

---

### DEBT-ID-CEILING-001 — Membership anchor & hierarchical scope ceiling (ADR-ID-ORG-005)

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore (+ Organization tree read) |
| **2. Section** | Holding Identity admin / Scope grant |
| **3. Reason** | Identity permissions without org ceiling allow peer/upward scope visibility and grant; privilege expansion risk. |
| **4. Layer type** | Architecture + Backend (+ FE pickers later) |
| **5. Created at** | 2026-10-17 |
| **6. Owner decision** | ADR-ID-ORG-005 ACCEPTED — laws L01–L07 |
| **7. Suggestion** | C0 home_company_id; C1–C4 HoldingAccessService + Scope list filter; C5 tests; C6 backfill |
| **Status** | **CLOSED** (2026-10-18) — BE complete on develop: home_company_id + C6 backfill, HierarchyCeilingSupport, HAS actorCeiling + L04 manage wire, ScopeService list filter, UserService home on create, HierarchyCeilingTest + HoldingDelegatedAdminTest green (6563bc2d). FE pickers residual deferred (no separate debt unless product asks). |

## Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.5 | 2026-10-14 | DEBT-ORG-012 OPEN |
| 1.6 | 2026-10-14 | ADR-FIN-003; FIN-HOLD debts; CoA OPEN |
| 1.7 | 2026-10-14 | **ADR-ID-ORG-004** + work breakdown; DEBT-ID-H01…H04 OPEN |
| 1.8 | 2026-10-15 | H01–H04 CLOSED; FIN-HOLD-001 CLOSED; HOLD-002 PARTIAL; HOLD-003 OPEN |
| 1.9 | 2026-10-16 | FIN-HOLD-003 PARTIAL after W3 BE+API |
| 2.0 | 2026-10-16 | FIN-HOLD-W4 progress |
| 2.1 | 2026-10-16 | FIN-HOLD residual notes |
| 2.2 | 2026-10-16 | FIN-HOLD-W5/W6 closed; HOLD-002 residual |
| **2.3** | **2026-10-16** | **HOLD-002 CLOSED** (grant = Identity role/scope assign). **DEBT-HOLD-W7 OPEN** registered. ORG-012 expanded. |
| **2.4** | **2026-10-17** | **ADR-ID-ORG-005** membership anchor & scope ceiling; **DEBT-ID-CEILING-001 PARTIAL**. |
| **2.5** | **2026-10-18** | **DEBT-ID-CEILING-001 CLOSED** — ceiling BE delivered + tests green on develop (6563bc2d). Debt Register restored after accidental wipe. |
