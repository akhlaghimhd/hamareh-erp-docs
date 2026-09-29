# ADR — Competitive Upgrade: Identity & Organization Layers v1.0

- **Document ID:** ADR-COMP-UPG-ID-ORG-v1.0
- **Status:** Accepted (product direction locked for execution)
- **Date:** 2026-09-29
- **Last progress update:** 2026-09-29 (Wave 1 closed for planned track)
- **Scope:** IdentityCore (Layer 4) + Organization (Layer 5) + Platform (SaaS Admin feature catalog)

---

## Progress (Wave 1) — CLOSED for planned BE track

| Code | Status | Evidence |
|------|--------|----------|
| ID-W1-05 | **DONE** | AuthController::changePassword, PasswordPolicyService, PasswordPolicyAndSetPasswordTest |
| ID-W1-06 | **DONE** | GET roles/user/{id}, assignRoleToUser full sync |
| ID-W1-01 | **DONE foundation** | tenant_sod_rules + RLS, SodService, SodController, SodRulesTest green |
| ID-W1-02 | **DONE foundation** | RoleService::assignRoleToUser → SodService::assertAssignable |
| PLT-W1-01 | **DONE** | platform_feature_catalog + tenant_feature_entitlements + FeatureCatalogService + 6 tests green |
| PLT-W1-02 | **DONE** | Company/Branch/BU/OrgHierarchy gates + FeaturePackGateTest |
| PLT-W1-03 | **DONE** | freezeEntitlement / unfreeze; freeze keeps data, blocks new create |
| ID-W1-03 | **DONE foundation** | TOTP MFA (native), recovery codes, login challenge mfa_challenge, MfaServiceTest 5 green |
| ID-W1-04 | **TODO (deferred)** | SSO OIDC/SAML — next competitive slice or Wave 2 parallel |
| ORG-W1 residual | deferred | FE hub cards gate by entitlement; pack UX |

### Residual closed
- PermissionSeeder: `identity.sod.view`, `identity.sod.manage`, `identity.mfa.manage` (ensureIdentityExtras)

### Residual open (non-blocking Wave 1)
- Optional demo SoD conflict rules for demo tenant
- FE: MFA setup UI + entitlement gate on hub cards
- ID-W1-04 SSO

### Next
- **ID-W1-04 SSO** (if closing Wave 1 fully) **or** start **Wave 2** (Access Certification, Privileged Access, IC Posting with Accounting)

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

## MFA law (foundation)

- TOTP (RFC 6238) native; recovery codes single-use.
- Login: if MFA confirmed → return `requires_mfa` + short-lived `mfa_challenge` token; full JWT only after verify.
