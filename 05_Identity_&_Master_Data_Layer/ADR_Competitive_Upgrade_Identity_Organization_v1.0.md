# ADR — Competitive Upgrade: Identity & Organization Layers v1.0

- **Document ID:** ADR-COMP-UPG-ID-ORG-v1.0
- **Status:** Accepted (product direction locked for execution)
- **Date:** 2026-09-29
- **Last progress update:** 2026-09-29
- **Scope:** IdentityCore (Layer 4) + Organization (Layer 5) + Platform (SaaS Admin feature catalog)

---

## Progress (Wave 1)

| Code | Status | Evidence |
|------|--------|----------|
| ID-W1-05 | **DONE** | AuthController::changePassword, PasswordPolicyService, tests PasswordPolicyAndSetPasswordTest |
| ID-W1-06 | **DONE** | GET roles/user/{id}, assignRoleToUser full sync |
| ID-W1-01 | **DONE foundation** | tenant_sod_rules + RLS, TenantSodRule, SodService, SodController, routes under identity/sod-rules |
| ID-W1-02 | **DONE foundation** | RoleService::assignRoleToUser calls SodService::assertAssignable before mutation |
| PLT-W1-01..03 | TODO | Feature catalog still required |
| ID-W1-03/04 | TODO | SSO / MFA |
| ORG-W1-01/02 | TODO | after catalog |

### Residual for SoD
- PermissionSeeder entries: `identity.sod.view`, `identity.sod.manage`
- PHPUnit SodRulesTest (file prepared locally; push if missing)
- Optional seed of sample conflict rules for demo tenant

### Code commits (hamarehSaasErp)
- 497b244 migration + model + SodService
- 99357e6 SodController + routes
- 93c8744 RoleService SoD hook restored

### Docs commits (hamareh-erp-docs)
- abd36df ADR v1.0
- a0b3e4f mark 05/06 DONE
- this update

**Next:** PLT-W1-01 Feature Catalog in SaasPlatform module, then PermissionSeeder for SoD codes.
