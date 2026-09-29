# ADR — Competitive Upgrade: Identity & Organization Layers v1.0

- **Status:** Accepted — Wave 1 CLOSED; Wave 2 Identity CLOSED; Wave 3 Identity foundation CLOSED
- **Last update:** 2026-09-29

## Wave 1 status (final)

| Code | Status |
|------|--------|
| PLT-W1-01 Catalog | DONE |
| PLT-W1-02 Org + Identity gates | DONE |
| PLT-W1-03 Freeze/Unfreeze | DONE |
| ID-W1-01/02 SoD | DONE foundation |
| ID-W1-03 SSO OIDC+SAML | DONE foundation |
| ID-W1-04 MFA | DONE foundation |
| ID-W1-05/06 | DONE |
| ORG-W1-01/02 | DONE via PLT-W1-02 |

## Wave 2 status (Identity foundation CLOSED)

| Code | Status |
|------|--------|
| ID-W2-01 Access Certification | DONE |
| ID-W2-02 Privileged / Emergency Access | DONE |
| ID-W2-03 Role Inheritance | DONE |
| ID-W2-04 Scope ↔ Hierarchy Purpose | DONE |
| ID-W2-05 Joiner / Mover / Leaver | DONE |
| ORG-W2 IC / Consol | BLOCKED on Accounting module |

## Wave 3 status (Identity foundation CLOSED)

| Code | Status |
|------|--------|
| ID-W3-01 Time-bounded role assignment | DONE |
| ID-W3-02 Role assignment approval workflow | DONE |
| ID-W3-03 Session / access re-evaluation | DONE |
| ID-W3-04 Identity audit export package | DONE |
| ID-W3-05 SCIM 2.0 Users foundation | DONE |

### Explicit residual (not blocking Wave 3 foundation)
- SCIM HTTP routes (`/scim/v2/*`) + bearer client-credentials auth
- Wire `IdentitySessionReevaluationService::invalidateUser` into all remaining mutators (RoleService.assign already partially caches; Approval approve wired)
- FE for Access Cert / Privileged / JML / Approval / SCIM admin
- SAML ACS assertion parse + OIDC JWKS verify
- Billing-driven pack upgrade events

### Next priorities
1. Residual wiring + SCIM routes (optional polish)
2. FE Identity advanced surfaces
3. ORG-W3 IC/Consol when Accounting module is ready
