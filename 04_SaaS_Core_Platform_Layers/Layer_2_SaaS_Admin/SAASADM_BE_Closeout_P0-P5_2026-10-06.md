# SAASADM Layer 2 — BE close-out status (2026-10-06)

**Related:** `02_System_Blueprint_&_Roadmaps/Project_Debt_Register_v1.0.md` · ADR-SAASADM-001

## Backend waves (hamarehSaasErp `develop`)

| Wave | Status | Notes |
|------|--------|-------|
| P0 Feature catalog + entitlement | CLOSED | L1 FeatureCatalogService |
| P1 Org FeaturePackGate | CLOSED (BE) | FE residual DEBT-ORG-008 |
| P2 Admin IAM | CLOSED | admin auth, RBAC, lockout, env seeder |
| P3 System settings | CLOSED (BE) | retention defaults; dual-approval = tenant_settings |
| P4 Audit | CLOSED | LOGIN / GRANT / REVOKE |
| P5 Tenant ops | CLOSED (BE) | GET /saas-admin/tenants (+ show) |
| P6 Notifications/Support | DEFERRED | skeleton exists |
| P7 FE admin shell | OPEN | DEBT-SAAS-001 |

## New residual debts

### DEBT-SAAS-001 — Frontend SaaS Admin shell + pack/tenant pages

| Field | Value |
|-------|--------|
| **1. Module** | SaasAdmin (L2) + Front |
| **2. Section** | Admin shell, feature catalog, entitlement per tenant, system settings UI |
| **3. Reason** | Backend APIs live; no `/admin` FE shell yet; tenant Org hub needs enabled_codes gating. |
| **4. Layer type** | Frontend |
| **5. Created at** | 2026-10-06T19:50:00+02:00 |
| **6. Owner decision** | ثبت پس از بسته شدن BE P0–P5 |
| **7. Suggestion** | Scaffold `/admin` separate from tenant dashboard; login `POST /api/v1/saas-admin/auth/login`; catalog, tenants, grant/revoke, system settings; i18n FA. Parallel: tenant FE hub cards (DEBT-ORG-008). |
| **Status** | OPEN |

### DEBT-SAAS-002 — Tenant-facing system settings page (dual-approval + retention)

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore + SaasAdmin + Front |
| **2. Section** | Tenant settings UX |
| **3. Reason** | Dual-approval in tenant_settings (correct); temp UI `/dashboard/identity/settings`. Platform retention defaults in system_settings; purge job open. |
| **4. Layer type** | Frontend (primary) + Backend purge job |
| **5. Created at** | 2026-10-06T19:50:00+02:00 |
| **6. Owner decision** | Move Identity temp box to formal tenant settings; do not put dual-approval in platform system_settings |
| **7. Suggestion** | Tenant settings under dashboard; IdentitySettingsController stays; `erp:purge-soft-deleted` using retention.* (DEBT-ORG-007 / ID-003). |
| **Status** | OPEN |

## PLT index (intended updates)

- **DEBT-PLT-001** → CLOSED BE (catalog + entitlement APIs)
- **DEBT-PLT-002** → PARTIAL (BE done; FE move of Identity settings open)
- **DEBT-PLT-003** → PARTIAL (platform retention keys seeded; tenant UX + job open)
