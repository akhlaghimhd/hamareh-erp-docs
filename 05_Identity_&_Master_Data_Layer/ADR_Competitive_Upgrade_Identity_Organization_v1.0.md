# ADR — Competitive Upgrade: Identity & Organization Layers v1.0

- **Status:** Accepted — Wave 1 CLOSED; Wave 2 Identity CLOSED; Wave 3 in progress
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

### Explicit deferred (Wave 2)
- FE for Access Cert / Privileged / JML / Scope purpose UX
- Billing-driven pack upgrade events
- SAML ACS assertion parse + OIDC JWKS verify

## Wave 3 — Advanced Identity controls

| Code | Title | Type | Status |
|------|-------|------|--------|
| ID-W3-01 | Time-bounded role assignment (valid_from / valid_to) | از صفر | IN PROGRESS |
| ID-W3-02 | Approval workflow for sensitive role assign | از صفر | TODO |
| ID-W3-03 | Session / continuous access re-evaluation hook | از صفر | TODO |
| ID-W3-04 | Audit export package (membership + role + privilege) | از صفر | TODO |
| ID-W3-05 | SCIM 2.0 provisioning foundation (enterprise IdP) | از صفر | TODO |
| ORG-W3-01 | Hierarchy validity windows enforced in Org services | ارتقا | TODO |
| ORG-W3-02 | IC / Consol (when Accounting ready) | از صفر | BLOCKED |

### Next
Complete ID-W3-01 (wire assign API + expire job optional), then ID-W3-02 or ID-W3-04.
