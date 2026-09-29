# ADR — Competitive Upgrade: Identity & Organization Layers v1.0

- **Document ID:** ADR-COMP-UPG-ID-ORG-v1.0
- **Status:** Accepted
- **Date:** 2026-09-29
- **Last progress update:** 2026-09-29 (Wave 1 fully closed including SSO foundation)
- **Scope:** IdentityCore + Organization + Platform feature catalog

---

## Progress (Wave 1) — CLOSED

| Code | Status | Evidence |
|------|--------|----------|
| ID-W1-05 | **DONE** | change-password + PasswordPolicy |
| ID-W1-06 | **DONE** | user-roles list + replace |
| ID-W1-01/02 | **DONE foundation** | SoD rules + assign hook |
| PLT-W1-01/02/03 | **DONE** | Feature catalog + org gates + freeze |
| ID-W1-03 | **DONE foundation** | TOTP MFA + challenge |
| ID-W1-04 | **DONE foundation** | OIDC providers, identity link, authorize/callback, SsoServiceTest |

### Wave 1 open residuals (non-blocking)
- JWKS signature verification hardening
- SAML protocol
- auto_provision + controlled membership policy
- FE MFA/SSO UX

### Next: Wave 2
- ID-W2-01 Access Certification
- ID-W2-02 Privileged Access
- ORG-W2-01/02 IC Posting + Consolidation (needs Accounting)

## SSO law (foundation)
- Per-tenant OIDC providers in `tenant_sso_providers` (RLS).
- External subject linked in `user_sso_identities`.
- Login requires existing tenant membership unless future auto_provision policy is enabled.
- MFA challenge still applies after SSO when enabled.
