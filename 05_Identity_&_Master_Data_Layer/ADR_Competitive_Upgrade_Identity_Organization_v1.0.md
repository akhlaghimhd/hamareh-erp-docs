# ADR — Competitive Upgrade: Identity & Organization Layers v1.0

- **Document ID:** ADR-COMP-UPG-ID-ORG-v1.0
- **Status:** Accepted (product direction locked for execution)
- **Date:** 2026-09-29
- **Last progress update:** 2026-09-29 (PLT-W1-02 green)
- **Scope:** IdentityCore (Layer 4) + Organization (Layer 5) + Platform (SaaS Admin feature catalog)

---

## Progress (Wave 1)

| Code | Status | Evidence |
|------|--------|----------|
| ID-W1-05 | **DONE** | AuthController::changePassword, PasswordPolicyService, PasswordPolicyAndSetPasswordTest |
| ID-W1-06 | **DONE** | GET roles/user/{id}, assignRoleToUser full sync |
| ID-W1-01 | **DONE foundation** | tenant_sod_rules + RLS, SodService, SodController, SodRulesTest green |
| ID-W1-02 | **DONE foundation** | RoleService::assignRoleToUser → SodService::assertAssignable |
| PLT-W1-01 | **DONE** | platform_feature_catalog + tenant_feature_entitlements + FeatureCatalogService + FeatureCatalogServiceTest (6 green) |
| PLT-W1-02 | **DONE** | Company/Branch/BU/OrgHierarchy gates + FeaturePackGateTest (6 green) |
| PLT-W1-03 | **IN PROGRESS** | Freeze on disable (is_enabled=false keeps data; create blocked by assertEnabled) |
| ID-W1-03/04 | TODO | SSO / MFA |
| ORG-W1 residual | deferred | pack UX on FE after freeze |

### Residual (non-blocking)
- PermissionSeeder: `identity.sod.view`, `identity.sod.manage`
- Optional demo SoD conflict rules for demo tenant
- FE: hub cards gate by entitlement codes

### Code commits (hamarehSaasErp) — selected
- SoD: 497b244, 99357e6, 93c8744, b1e8cb2 (SodRulesTest)
- Feature Catalog: 993717a, 34b71a4
- Org feature gates: 313664e, 0cfdce4, 2f14027, 11e3b0f (OrgHierarchy restore + CUSTOM gate)

### Docs commits (hamareh-erp-docs)
- abd36df ADR v1.0 accepted
- this update — PLT-W1-01/02 DONE

**Next:** close PLT-W1-03 freeze semantics explicitly, then ID-W1-03 MFA foundation (or residual seeder).

---

## Product law (Feature packs — independent)

- `multi_company`, `multi_branch`, `multi_business_unit`, `custom_org_hierarchy` are **independent** sellable packs.
- Missing entitlement = **disabled** (must purchase).
- Disable pack = **freeze**: existing rows remain readable; **create** of additional entities is blocked via `FeatureCatalogService::assertEnabled`.
- Primary company + default HQ branch remain available without packs (onboarding path).

## SoD law (foundation)

- Pairwise role conflicts in `tenant_sod_rules` (tenant-scoped + RLS).
- Enforcement modes: `BLOCK` | `WARN`.
- Hook: before `RoleService::assignRoleToUser` mutation.
