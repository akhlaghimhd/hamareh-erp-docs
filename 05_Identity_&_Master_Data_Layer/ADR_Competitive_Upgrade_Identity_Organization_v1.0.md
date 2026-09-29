# ADR — Competitive Upgrade: Identity & Organization Layers v1.0

- **Document ID:** ADR-COMP-UPG-ID-ORG-v1.0
- **Status:** Accepted (product direction locked for execution)
- **Date:** 2026-09-29
- **Last progress update:** 2026-09-29
- **Scope:** IdentityCore (Layer 4) + Organization (Layer 5) + Platform (SaaS Admin feature catalog)
- **Related:** ORG_Layer_Completion_Roadmap_v1.0, ORG_Smart_Hierarchy_Product_Law_v1.0, ORG_Smart_Hierarchy_Status_and_Debt_v1.0, ADR-ORG-001, ADR-ORG-002, Database Layer 4 - Identity & Access Core
- **Code repos:** akhlaghimhd/hamarehSaasErp, akhlaghimhd/hamarehSaasErp-Front
- **Docs repo:** akhlaghimhd/hamareh-erp-docs

---

## 1. Context & Decision

After competitive analysis against SAP S/4HANA, Oracle Fusion Cloud ERP, Microsoft Dynamics 365 Finance & Operations, Oracle NetSuite OneWorld and peer systems (2026), the following gaps were identified in Identity and Organization surfaces.

**Decision:** All items in Waves 1–3 are **MUST** for enterprise competitiveness. Sequencing is by dependency and commercial impact. No simplification that reduces the locked multi-tenant + RLS + Scope + feature-pack model is allowed.

**Non-negotiable constraints (unchanged):**
- Tenant isolation by design (`tenant_id` + PostgreSQL RLS ENABLE+FORCE)
- Soft delete + `row_version` on operational tables
- No physical FK across module boundaries (logical UUID only)
- Feature packs remain independent (`multi_company`, `multi_branch`, `multi_business_unit`, future `custom_org_hierarchy`)
- Hierarchy remains derived map (auto-sync); end-user rebuild is not primary UX

---

## 2. Coding scheme

| Prefix | Meaning |
|--------|--------|
| `ID-W1-xx` | Identity – Wave 1 (Immediate) |
| `ID-W2-xx` | Identity – Wave 2 (High) |
| `ID-W3-xx` | Identity – Wave 3 (Enterprise completion) |
| `ORG-W1-xx` | Organization – Wave 1 |
| `ORG-W2-xx` | Organization – Wave 2 |
| `ORG-W3-xx` | Organization – Wave 3 |
| `PLT-W1-xx` | Platform / SaaS Admin – Wave 1 |
| `PLT-W2-xx` | Platform – Wave 2 |

Status values: `TODO` | `IN_PROGRESS` | `DONE` | `BLOCKED`

---

## 3. Wave 1 — Immediate (make platform sellable to serious customers)

### Platform

| Code | Title | Type | Owner | Status | Notes |
|------|-------|------|-------|--------|-------|
| PLT-W1-01 | SaaS Admin Feature Catalog (storage + API) | New | Platform | TODO | Store purchased packs per tenant; source of truth for all gating |
| PLT-W1-02 | Feature-pack enforcement (BE + FE) | Upgrade | Platform + Org + Identity | TODO | Replace env flags; enforce on every Org/Identity surface |
| PLT-W1-03 | Pack upgrade/downgrade behaviour | New | Platform + Org | TODO | On enable → ensureStructuralTrees; on disable → hide/freeze (never hard-delete). Closes D1 + D7 |

### Identity

| Code | Title | Type | Owner | Status | Notes |
|------|-------|------|-------|--------|-------|
| ID-W1-01 | SoD Matrix (role conflict rules + risk detection) | New | IdentityCore | IN_PROGRESS | Conflict pairs, severity, mitigation; report API |
| ID-W1-02 | SoD evaluation on role assign | New | IdentityCore | TODO | Block or warn on conflicting assignment |
| ID-W1-03 | SSO (OIDC + SAML) foundation | New | IdentityCore | TODO | Login via external IdP; map claims to tenant_user |
| ID-W1-04 | MFA (TOTP + recovery) | New | IdentityCore | TODO | Optional per tenant/user; enforce on sensitive actions |
| ID-W1-05 | Change-password endpoint + Credential Policy | Upgrade | IdentityCore | **DONE** | Already in code: AuthController::changePassword, PasswordPolicyService, setPassword, forgot-password confirm; tests PasswordPolicyAndSetPasswordTest. Residual: password-history store (optional later). |
| ID-W1-06 | Full user-roles read API + replace semantics | Upgrade | IdentityCore | **DONE** | Already in code: GET roles/user/{userId}, RoleService::listRolesForUser, assignRoleToUser does full sync (add+remove). |

### Organization (Wave 1 support)

| Code | Title | Type | Owner | Status | Notes |
|------|-------|------|-------|--------|-------|
| ORG-W1-01 | Respect feature catalog on all Org surfaces | Upgrade | Organization | TODO | Hub cards, lists, create flows, APIs gated by packs |
| ORG-W1-02 | CUSTOM hierarchy pack flag wiring | Upgrade | Organization | TODO | Replace FEATURE_CUSTOM_ORG_HIERARCHY env with catalog flag |

**Wave 1 exit criteria:** Feature catalog live; SoD basic rules enforceable; SSO+MFA login path exists; change-password + full role list closed; Org UI respects packs.

---

## 4. Wave 2 — High priority (compete with NetSuite / Dynamics 365)

### Identity

| Code | Title | Type | Owner | Status | Notes |
|------|-------|------|-------|--------|-------|
| ID-W2-01 | Access Certification / Review campaigns | New | IdentityCore | TODO | Periodic campaigns; manager attestation; revoke workflow |
| ID-W2-02 | Privileged / Emergency Access (time-boxed) | New | IdentityCore | TODO | Elevate with expiry + full audit trail |
| ID-W2-03 | Role derivation / composite roles + live inheritance | Upgrade | IdentityCore | TODO | Parent role permission inheritance (snapshot still allowed) |
| ID-W2-04 | Scope ↔ Hierarchy purpose binding | Upgrade | IdentityCore + Org | TODO | Scope resolution can use hierarchy nodes |
| ID-W2-05 | MembershipHistory → full Joiner/Mover/Leaver hooks | Upgrade | IdentityCore | TODO | Events + optional auto-provisioning hooks |

### Organization

| Code | Title | Type | Owner | Status | Notes |
|------|-------|------|-------|--------|-------|
| ORG-W2-01 | Intercompany posting + elimination engine (contract + events) | New | Org + Accounting | TODO | Depends on Accounting readiness; extend ADR-ORG-002 |
| ORG-W2-02 | Consolidation run (snapshot, rate set ref, CTA placeholder) | New | Org + Accounting | TODO | Org owns hierarchy snapshot; Accounting owns numbers |
| ORG-W2-03 | Hierarchy report consumers wired to HierarchyReportContract | Upgrade | Org + consumers | TODO | Close D2 consumer side |
| ORG-W2-04 | BUSINESS_UNIT scope fully enforced | Upgrade | Identity + Org | TODO | Already prepared; complete middleware + tests |
| ORG-W2-05 | Ownership effective % + minority interest helpers | Upgrade | Organization | TODO | Calculation service |

**Wave 2 exit criteria:** SoD + Certification usable; Emergency Access live; IC/Consol contracts green; Scope hierarchy-aware; packs fully enforced.

---

## 5. Wave 3 — Enterprise completion (close gap to SAP/Oracle class)

### Identity

| Code | Title | Type | Owner | Status | Notes |
|------|-------|------|-------|--------|-------|
| ID-W3-01 | ABAC / Policy Decision Point on Scope | New | IdentityCore | TODO | Attributes (time, IP, device, custom) evaluated at request |
| ID-W3-02 | SCIM 2.0 provisioning endpoint | New | IdentityCore | TODO | Create/update/deactivate from external IdP/HR |
| ID-W3-03 | Adaptive / risk-based authentication | New | IdentityCore | TODO | Build on MFA + context |
| ID-W3-04 | Advanced audit & access analytics dashboard | New | IdentityCore | TODO | Who has what, last used, orphaned roles |

### Organization

| Code | Title | Type | Owner | Status | Notes |
|------|-------|------|-------|--------|-------|
| ORG-W3-01 | TAX + MANAGEMENT hierarchies first-class (SYS trees optional) | Upgrade | Organization | TODO | Catalog already exists; make report-grade |
| ORG-W3-02 | Profit Center / Segment dimension + reporting hooks | New | Organization | TODO | Align with Cost Center; logical to Accounting |
| ORG-W3-03 | Sales Org — Distribution Channel + Division | Upgrade | Organization | TODO | Complete competitive sales structure |
| ORG-W3-04 | Enterprise Structure Configurator (template apply) | Upgrade | Organization | TODO | ORG-P6-06 full productisation |
| ORG-W3-05 | Value Stream / Work Center level (manufacturing readiness) | New | Organization | TODO | Only if manufacturing track starts |

**Wave 3 exit criteria:** Policy-based access live; SCIM working; Tax/Management hierarchies reportable; Sales Org complete; configurator usable for large onboarding.

---

## 6. Explicit non-goals (do not reopen without new ADR)

- Cross-tenant trading (already abandoned in ADR-ORG-002)
- Hard-delete of hierarchy rows on pack downgrade
- Making Hierarchy the only source of Scope / authorization
- Physical FK to Accounting or Inventory tables
- Replacing PostgreSQL RLS with application-only isolation

---

## 7. Dependency summary

```
PLT-W1-01/02/03  ──►  ORG-W1-01/02, ID surfaces gating
ID-W1-01/02      ──►  ID-W2-01 (certification uses SoD risk)
ID-W1-03/04      ──►  ID-W3-02/03 (SCIM + Adaptive)
ORG-W2-01/02     ──►  Accounting module readiness
ID-W2-04         ──►  ORG hierarchy purpose catalog (already DONE foundation)
```

---

## 8. Execution rules

1. Backend-first for every code; FE after API acceptance.
2. Every new table: `tenant_id`, soft delete, `row_version`, RLS under `app_user`.
3. Tests must use PHPUnit 11 Attributes (`#[Test]`); isolation tests mandatory.
4. PermissionSeeder must be updated for every new permission code; demo owners re-seeded.
5. Docs updated in same PR or immediately after (this ADR is the living worklist).
6. Commits English; user-facing reports Persian.

---

## 9. Progress log

| Date | Codes | Action |
|------|-------|--------|
| 2026-09-29 | — | ADR v1.0 accepted |
| 2026-09-29 | ID-W1-05, ID-W1-06 | Marked **DONE** after code audit (change-password + policy + user-roles list/replace already shipped) |
| 2026-09-29 | ID-W1-01 | Started SoD matrix implementation |

**Next active work:** ID-W1-01 SoD schema + service + evaluation hook (ID-W1-02), then PLT-W1-01 Feature Catalog.

---

## 10. Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-09-29 | Initial competitive upgrade worklist; Waves 1–3 locked |
| 1.0.1 | 2026-09-29 | Progress: ID-W1-05/06 DONE; ID-W1-01 IN_PROGRESS |

**Citation name:** `ADR_Competitive_Upgrade_Identity_Organization_v1.0`  
**Path:** `05_Identity_&_Master_Data_Layer/ADR_Competitive_Upgrade_Identity_Organization_v1.0.md`
