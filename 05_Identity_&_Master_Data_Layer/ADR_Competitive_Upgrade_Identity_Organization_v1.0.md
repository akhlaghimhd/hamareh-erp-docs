# ADR — Competitive Upgrade: Identity & Organization Layers v1.0

- **Status:** Accepted — Wave 1 CLOSED including residuals
- **Last update:** 2026-09-29

## Wave 1 status (final)

| Code | Status |
|------|--------|
| PLT-W1-01 Catalog | DONE |
| PLT-W1-02 Org + Identity gates | DONE (Org create + Identity Scope BUSINESS_UNIT) |
| PLT-W1-03 Freeze/Unfreeze | DONE (service + API endpoints + unfreeze test) |
| ID-W1-01/02 SoD | DONE foundation |
| ID-W1-03 SSO OIDC+SAML | DONE foundation (OIDC full path; SAML AuthnRequest begin; ACS assertion parse deferred) |
| ID-W1-04 MFA | DONE foundation |
| ID-W1-05/06 | DONE |
| ORG-W1-01/02 | DONE via PLT-W1-02 |

### Explicit deferred (not blocking Wave 1)
- SAML ACS assertion parse + signature
- OIDC JWKS signature verification
- Billing-driven auto upgrade from subscription events
- FE pack/SSO/MFA UX

### Next: Wave 2
ID-W2-01 Access Certification · ID-W2-02 Privileged Access · ORG-W2 IC/Consol (Accounting)
