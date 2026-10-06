# Project Debt Register v1.0

**Document ID:** DEBT-REGISTER-v1.0  
**SSOT for open project debts across all layers/modules**  
**Created:** 2026-10-06  
**Last updated:** 2026-10-06T19:55:00+02:00  
**Repos:** akhlaghimhd/hamareh-erp-docs  

> **Restore note (2026-10-06):** Full historical entries (DEBT-ID-001…011, DEBT-ORG-001…011) remain in git history at commit `cf30dd8`. This file was briefly corrupted by an incomplete push and is republished with current SAASADM close-out + pointer to history.

**Related:**
- `04_SaaS_Core_Platform_Layers/Layer_2_SaaS_Admin/SAASADM_BE_Closeout_P0-P5_2026-10-06.md`
- `04_SaaS_Core_Platform_Layers/Layer_2_SaaS_Admin/ADR-SAASADM-001_Feature_Pack_Model_v1.0.md`
- Full prior body: `git show cf30dd8:02_System_Blueprint_&_Roadmaps/Project_Debt_Register_v1.0.md`

---

## Register law (LOCKED 2026-10-06)

Every new debt entry **must** include: Module, Section, Reason, Layer type, Created at, Owner decision, Suggestion.  
Closed debts: mark **Status: CLOSED** (do not delete).  
Status values: OPEN | BLOCKED | PARTIAL | DEFERRED | CLOSED

---

## C. Cross-cutting / Platform blockers (index)

| Debt ID | Title | Status |
|---------|-------|--------|
| DEBT-PLT-001 | SaaS Admin feature catalog + pack purchase API | **CLOSED BE** — L1 FeatureCatalog + admin entitlement APIs (ADR-SAASADM-001). FE shell = DEBT-SAAS-001 |
| DEBT-PLT-002 | System settings page (move identity dual-approval UI) | **PARTIAL** — BE tenant_settings correct; FE move open |
| DEBT-PLT-003 | Tenant retention setting UI | **PARTIAL** — platform retention.* keys seeded; purge job + tenant UX open |

---

## E. Layer 2 — SaaS Admin — BE P0–P5 CLOSED (2026-10-06)

| Wave | Status |
|------|--------|
| P0–P5 Backend | CLOSED on `hamarehSaasErp` develop |
| P6 Notifications/Support | DEFERRED (skeleton exists) |
| P7 FE admin shell | OPEN — DEBT-SAAS-001 |

### DEBT-SAAS-001 — Frontend SaaS Admin shell

| Field | Value |
|-------|--------|
| **1. Module** | SaasAdmin (L2) + Front |
| **2. Section** | Admin shell, catalog, entitlement, system settings UI |
| **3. Reason** | Backend APIs live; no `/admin` FE shell |
| **4. Layer type** | Frontend |
| **5. Created at** | 2026-10-06T19:50:00+02:00 |
| **6. Owner decision** | Next FE priority after BE P0–P5 |
| **7. Suggestion** | `/admin` routes; login `POST /api/v1/saas-admin/auth/login`; tenants + grant/revoke; i18n FA |
| **Status** | OPEN |

### DEBT-SAAS-002 — Tenant settings page (dual-approval + retention)

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore + SaasAdmin + Front |
| **2. Section** | Tenant settings UX |
| **3. Reason** | Dual-approval in tenant_settings; temp UI under Identity; retention keys on platform only |
| **4. Layer type** | Frontend + Backend purge job |
| **5. Created at** | 2026-10-06T19:50:00+02:00 |
| **6. Owner decision** | Formal tenant settings page; not platform system_settings for dual-approval |
| **7. Suggestion** | Move from `/dashboard/identity/settings`; implement purge job (DEBT-ORG-007 / ID-003) |
| **Status** | OPEN |

### Org pack residuals (unchanged intent)

| Debt ID | Status |
|---------|--------|
| DEBT-ORG-001 | PARTIAL — BE CLOSED; FE residual |
| DEBT-ORG-002 | CLOSED (BE) |
| DEBT-ORG-008 | PARTIAL — BE CLOSED; FE hub cards residual |
| DEBT-ID-009 | PARTIAL — catalog SoT exists; H4/H5 Holding open |

**Restore full ID/ORG debt tables from history when capacity allows** (`cf30dd8`).

---

## Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-10-06 | Initial register |
| 1.1 | 2026-10-06 | SAASADM-P0/P1 BE verified |
| 1.2 | 2026-10-06 | SAASADM-P2–P5 BE closed; DEBT-SAAS-001/002; temporary compact restore after push regression |
