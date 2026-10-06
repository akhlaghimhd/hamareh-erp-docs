# Project Debt Register v1.0

**Document ID:** DEBT-REGISTER-v1.0  
**SSOT for open project debts across all layers/modules**  
**Created:** 2026-10-06  
**Last updated:** 2026-10-06T11:00:00+02:00  
**Repos:** akhlaghimhd/hamareh-erp-docs  
**Related status docs (do not duplicate; link only):**
- `05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/ORG_Smart_Hierarchy_Status_and_Debt_v1.0.md`
- `05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/ORG_Intercompany_Status_and_Debt_v1.0.md`
- `05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/ORG_Sales_Purch_Status_and_Debt_v1.0.md`
- `04_SaaS_Core_Platform_Layers/Layer_2_SaaS_Admin/ADR-SAASADM-001_Feature_Pack_Model_v1.0.md`

---

## Register law (LOCKED 2026-10-06)

Every new debt entry **must** include all of the following fields, in this order:

1. **Module** — official layer/module (e.g. IdentityCore, Organization)
2. **Section** — sub-area inside the module
3. **Reason** — why the debt was created (checkable when closing)
4. **Layer type** — Backend | Frontend | Architecture (or combination)
5. **Created at** — ISO date-time of registration
6. **Owner decision** — explicit product/engineering decision (or «ثبت اولیه — منتظر تصمیم»)
7. **Suggestion** — recommended approach / unblock path

**Rules:**
- Do not delete closed debts; mark **Status: CLOSED** with close date and verification note.
- Prefer linking to existing per-topic Status_and_Debt files for deep detail; this register is the **index + decision log**.
- When user says «بدهی را ثبت کن», append here with the 7 fields above.
- Commit messages English; narrative for user in Persian.

**Status values:** OPEN | BLOCKED | PARTIAL | DEFERRED | CLOSED

---

## A. Layer 4 — Identity & Access Core (`IdentityCore`)

### DEBT-ID-001 — Change-password backend endpoint (T05)

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore (L4) |
| **2. Section** | Profile / Credential — change password |
| **3. Reason** | FE profile-me already has change-password UI; backend endpoint was deferred so password change cannot complete end-to-end. Need a secure authenticated endpoint (current password + new + confirmation) with rate limit and audit. |
| **4. Layer type** | Backend (primary) + Frontend wiring if needed |
| **5. Created at** | 2026-09-14 (discovered FE-P1); registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — قابل بستن در sprint کوتاه Identity close-out |
| **7. Suggestion** | Add `POST /api/v1/identity/profile/change-password` (or under credentials) with current_password, password, password_confirmation; invalidate other sessions optionally; test + Permission if any. Keep FE form already on profile-me. |
| **Status** | OPEN |

---

### DEBT-ID-002 — Dual role-approval UX polish

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore (L4) |
| **2. Section** | Role assignment / Dual approval queue |
| **3. Reason** | Backend dual-approval (require_role_assignment_approval) is done; FE still needs: toast distinguishing pending vs direct grant; member detail dimmed pending add/remove; rejected visible + re-request; no re-edit after approve/reject; ensure `identity.role.approve` in PermissionSeeder with clear FA label. Temp settings UI at `/dashboard/identity/settings` must later move to system settings. |
| **4. Layer type** | Frontend (primary) + Backend PermissionSeeder label check |
| **5. Created at** | 2026-10-02; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — UX debt; product rules for role-assignment-requests page already LOCKED |
| **7. Suggestion** | Minimal FE diffs only (UI safety law). Do not rewrite list pages. Confirm PermissionSeeder has `identity.role.approve` FA label. Plan move of tenant setting toggle to future system settings page (track as separate debt if needed). |
| **Status** | OPEN |

---

### DEBT-ID-003 — Soft-delete retention / purge for Identity lists

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore (L4) |
| **2. Section** | Soft-delete lifecycle — members, roles, scopes (deleted bucket) |
| **3. Reason** | Soft-delete retention/purge LAW locked 2026-10-05. Physical purge only via retention policy + scheduled job with referential guards. P0+P1 targeted Org masters first; Identity list pages with deleted bucket still lack retention UX + purge wiring. |
| **4. Layer type** | Backend (job + setting) + Frontend (retention UX on deleted bucket) |
| **5. Created at** | 2026-10-05; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — apply after Org companies purge pattern is proven |
| **7. Suggestion** | Reuse `erp:purge-soft-deleted` pattern + tenant retention setting; add Identity entity types with guards (no purge of last owner, active assignments, etc.). Remind when touching members/roles/scopes list pages. |
| **Status** | OPEN |

---

### DEBT-ID-004 — TenantCache deeper use (P4-X2)

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore (L4) |
| **2. Section** | Caching — TenantCache |
| **3. Reason** | TenantCache foundation done (F7); deeper adoption across Role/Permission/Scope hot paths not fully audited. Risk of stale or missing cache invalidation on rare paths. |
| **4. Layer type** | Backend |
| **5. Created at** | 2026-08-31; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — non-blocking for L4 close; schedule as performance hardening |
| **7. Suggestion** | Audit RoleService / ScopeService / AuthenticationService for remaining uncached reads; ensure invalidation on CRUD and assignment events. |
| **Status** | OPEN |

---

### DEBT-ID-005 — SoftDeletes / row_version consistency final check

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore (L4) |
| **2. Section** | Data standards — SoftDeletes + row_version |
| **3. Reason** | Layer 4 open note (2026-08-31): final consistency check across identity tables for SoftDeletes and row_version vs Law 1.4 / 1.5. |
| **4. Layer type** | Backend + Architecture |
| **5. Created at** | 2026-08-31; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — audit-only task before declaring L4 sealed |
| **7. Suggestion** | Script or test suite asserting soft-delete columns and row_version on all operational Identity tables; fix any gap without schema redesign. |
| **Status** | OPEN |

---

### DEBT-ID-006 — PermissionSeeder completeness (membership_history.view and new perms)

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore (L4) |
| **2. Section** | Permission catalog / PermissionSeeder |
| **3. Reason** | Historical note: PermissionSeeder may need `identity.membership_history.view` and any perms added later (e.g. role.approve FA label). Demo owners must receive grants; re-seed + re-login required after changes. |
| **4. Layer type** | Backend |
| **5. Created at** | 2026-08-31; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — include in Identity close-out |
| **7. Suggestion** | Diff PermissionSeeder against all gates used in Identity routes/controllers; add missing codes with FA labels; seed tenant-admin + owner grants. |
| **Status** | OPEN |

---

### DEBT-ID-007 — GET user-roles list API (read path)

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore (L4) |
| **2. Section** | Member roles — read API |
| **3. Reason** | Known debt 2026-09-15: no dedicated backend GET user-roles list; assign is append-only via POST. Member detail roles section historically placeholder until read API. Holding-aware constraints must apply. |
| **4. Layer type** | Backend (primary) + Frontend consumer |
| **5. Created at** | 2026-09-15; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — verify current state; if still missing, implement under HoldingAccessService constraints |
| **7. Suggestion** | `GET .../tenant-users/{id}/roles` returning active + pending (if dual-approval on); filter by assertCanManageUserId. FE member detail already partially wired. |
| **Status** | OPEN |

---

### DEBT-ID-008 — FE URL law for Identity detail routes

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore (L4) + Front |
| **2. Section** | Routing — members / roles detail |
| **3. Reason** | FE URL law 2026-10-05: no DB ids in browser path. Company detail already uses focus+detail pattern. DEBT: same pattern for members, roles, etc. |
| **4. Layer type** | Frontend |
| **5. Created at** | 2026-10-05; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — align when capacity allows; not blocking L4 backend seal |
| **7. Suggestion** | Mirror company pattern: fixed `/dashboard/identity/members/detail` + sessionStorage focus id; redirect legacy `/members/[id]`. |
| **Status** | OPEN |

---

### DEBT-ID-009 — Holding Access H4–H5 (pack + SaaS Admin)

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore (L4) + cross-cutting Holding |
| **2. Section** | ADR-ID-ORG-003 phases H4–H5 |
| **3. Reason** | H1–H3 done on main (HoldingAccessService, scopes, search, delegated admin tests). H4/H5 depend on SaaS Admin feature catalog and pack upgrade/downgrade. |
| **4. Layer type** | Architecture + Backend + SaaS Admin |
| **5. Created at** | 2026-10-02; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | 2026-10-06: L1 FeatureCatalog + entitlements live; Holding soft path remains until explicit H4/H5 product rules wired to packs. |
| **7. Suggestion** | Wire Holding soft/hard paths to purchased flags via FeatureCatalogService; do not invent parallel pack tables in Identity. |
| **Status** | PARTIAL — catalog/entitlement SoT exists (ADR-SAASADM-001); H4/H5 Holding product rules still open |

---

### DEBT-ID-010 — Optional: Privileged request dialog + Access-cert item certify UI

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore (L4) |
| **2. Section** | Privileged Access / Access Certification FE |
| **3. Reason** | Backend services and campaigns exist; optional FE polish: privileged request dialog, access-cert item certify UI. Not required for L4 architectural close. |
| **4. Layer type** | Frontend |
| **5. Created at** | 2026-09-29; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — optional / low priority |
| **7. Suggestion** | Schedule only after dual-approval UX and change-password are done. |
| **Status** | DEFERRED |

---

### DEBT-ID-011 — Enterprise SSO + SCIM (gap vs market)

| Field | Value |
|-------|--------|
| **1. Module** | IdentityCore (L4) |
| **2. Section** | Enterprise identity federation |
| **3. Reason** | Competitive gap vs NetSuite/Dynamics/SAP: no SAML/OIDC SSO or SCIM directory sync. Needed for upper mid-market sales. |
| **4. Layer type** | Architecture + Backend |
| **5. Created at** | 2026-10-06T09:53:00+02:00 (from competitive status report) |
| **6. Owner decision** | ثبت اولیه — out of current L4 close scope; track for enterprise readiness phase |
| **7. Suggestion** | Prefer integration layer (WorkOS-style) over building SAML from scratch; ADR when starting. |
| **Status** | DEFERRED |

---

## B. Layer 5 — Organization (`Organization`)

### DEBT-ORG-001 — Feature-pack upgrade/downgrade (Smart Hierarchy D1)

| Field | Value |
|-------|--------|
| **1. Module** | Organization (L5) |
| **2. Section** | Smart Hierarchy — pack-driven structural trees |
| **3. Reason** | Product law requires multi_company / multi_branch packs to drive hierarchy UX. |
| **4. Layer type** | Architecture + Backend + Frontend |
| **5. Created at** | 2026-09-25; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | 2026-10-06: Backend freeze/unfreeze + create gates done (FeatureCatalogService + FeaturePackGateTest). Residual: FE hub cards + ensureStructuralTrees consumer on granted event. |
| **7. Suggestion** | FE read enabled_codes from /feature-entitlements; on grant event ensureStructuralTrees; on freeze hide/freeze trees only. |
| **Status** | PARTIAL — BE gate + freeze/unfreeze CLOSED; FE residual |

---

### DEBT-ORG-002 — CUSTOM hierarchy pack in SaaS catalog (D7)

| Field | Value |
|-------|--------|
| **1. Module** | Organization (L5) |
| **2. Section** | Smart Hierarchy — CUSTOM trees |
| **3. Reason** | CUSTOM must be sellable pack independent of multi_company/multi_branch. |
| **4. Layer type** | Architecture + Backend + Frontend |
| **5. Created at** | 2026-09-25; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | 2026-10-06: Backend uses FeatureCatalogService::CODE_CUSTOM_ORG_HIERARCHY (assertEnabled on createHierarchy). Env gate removed from BE path. FE may still have NEXT_PUBLIC residual. |
| **7. Suggestion** | Remove any remaining FE env fallback; gate hub card by enabled_codes. |
| **Status** | CLOSED (BE) — FE residual tracked under DEBT-ORG-008 |

---

### DEBT-ORG-003 — Sales/Purch feature-pack gating (H3)

| Field | Value |
|-------|--------|
| **1. Module** | Organization (L5) |
| **2. Section** | Sales / Purchasing structure |
| **3. Reason** | Sales/Purch structure track closed for masters + H1–H2. H3 feature-pack gating still needs explicit pack codes on create paths. |
| **4. Layer type** | Backend + Frontend |
| **5. Created at** | 2026-09-29; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — still needs explicit pack codes + assertEnabled on Sales/Purch create paths |
| **7. Suggestion** | Gate hub cards and APIs by purchased packs; keep masters intact when pack off. |
| **Status** | OPEN |

---

### DEBT-ORG-004 — Sales-module consumers (H4)

| Field | Value |
|-------|--------|
| **1. Module** | Organization (L5) → future Sales (L6) |
| **2. Section** | Sales structure consumption |
| **3. Reason** | Org sales/purch masters ready; commercial document engines do not exist yet. H4 is consumer wiring, not Org redesign. |
| **4. Layer type** | Architecture (cross-module) |
| **5. Created at** | 2026-09-29; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — DEFERRED until Sales module |
| **7. Suggestion** | When Sales starts, bind offices/groups via Org assignment APIs; do not duplicate structure tables. |
| **Status** | DEFERRED |

---

### DEBT-ORG-005 — Intercompany Accounting / Ops phases (parked)

| Field | Value |
|-------|--------|
| **1. Module** | Organization (L5) + Accounting / Sales |
| **2. Section** | Intercompany post–P1 |
| **3. Reason** | Org-IC-P1 CLOSED (partners/rules). Acc-IC and Ops-IC parked until GL and SO/PO exist. |
| **4. Layer type** | Architecture + Backend (future modules) |
| **5. Created at** | 2026-09-28; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — PARKED; do not re-open Org IC tables |
| **7. Suggestion** | Resume only with Accounting GL green; implement Acc-IC against existing partner map; enforce `org.intercompany` pack (already gated on IC rule create). |
| **Status** | DEFERRED |

---

### DEBT-ORG-006 — Company ownership HTTP routes

| Field | Value |
|-------|--------|
| **1. Module** | Organization (L5) |
| **2. Section** | Company ownership (erp_company_ownerships) |
| **3. Reason** | Wave P1 delivered service + model + RLS; Ownership HTTP routes still deferred. |
| **4. Layer type** | Backend |
| **5. Created at** | 2026-09-23; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — open for close-out if FE needs ownership UI |
| **7. Suggestion** | Thin controller over CompanyOwnershipService; permission-gated; include in Organization API tests. |
| **Status** | OPEN |

---

### DEBT-ORG-007 — Soft-delete retention / purge for Org masters (P0 path)

| Field | Value |
|-------|--------|
| **1. Module** | Organization (L5) |
| **2. Section** | Soft-delete lifecycle — companies, branches, departments |
| **3. Reason** | LAW 2026-10-05: purge only via retention + job. P0+P1 target: tenant retention setting + `erp:purge-soft-deleted` for non-primary company, branch, department. Companies page first. |
| **4. Layer type** | Backend + Frontend |
| **5. Created at** | 2026-10-05; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — priority for Org foundation close |
| **7. Suggestion** | Implement job with referential guards (no purge primary, no purge with dependents without cascade rules already defined). FE retention settings + deleted-bucket messaging. |
| **Status** | OPEN |

---

### DEBT-ORG-008 — Platform enforce org.* feature packs on API + hub cards

| Field | Value |
|-------|--------|
| **1. Module** | Organization (L5) + SaaS Admin |
| **2. Section** | Feature packs runtime (multi_company, multi_branch, multi_business_unit, org.intercompany, …) |
| **3. Reason** | Product law locked: every Org UI/API surface must gate by purchased flags. |
| **4. Layer type** | Backend + Frontend + Architecture |
| **5. Created at** | 2026-09-25; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | 2026-10-06: BE create paths gated (CompanyService, BranchService, BusinessUnitService, OrgHierarchyService, IntercompanyService) + FeaturePackGateTest green. Residual: FE hub cards hide/show from enabled_codes. |
| **7. Suggestion** | FE: read GET /feature-entitlements; hide hub cards when pack off. |
| **Status** | PARTIAL — BE CLOSED; FE residual |

---

### DEBT-ORG-009 — Hierarchy large-tenant async rebuild tuning (D5 residual)

| Field | Value |
|-------|--------|
| **1. Module** | Organization (L5) |
| **2. Section** | Smart Hierarchy — rebuild performance |
| **3. Reason** | RebuildSystemHierarchiesJob + async path exist; full large-tenant performance tuning remains ops debt. |
| **4. Layer type** | Backend |
| **5. Created at** | 2026-09-29; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — non-blocking |
| **7. Suggestion** | Load-test with multi-company tenants; chunk node upserts; monitor queue. |
| **Status** | OPEN |

---

### DEBT-ORG-010 — FE Org detail panels restore / URL law for branches

| Field | Value |
|-------|--------|
| **1. Module** | Organization (L5) Front |
| **2. Section** | Company/Branch detail UX |
| **3. Reason** | Full company-detail UI temporarily simplified on develop; restore rich panels from prior when capacity allows. |
| **4. Layer type** | Frontend |
| **5. Created at** | 2026-10-05; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — restore from git history preferred over rewrite (UI safety law) |
| **7. Suggestion** | git log / restore known-good commits; do not full-rewrite list pages. |
| **Status** | OPEN |

---

### DEBT-ORG-011 — AriaSanatDemoOrgSeeder richness (optional)

| Field | Value |
|-------|--------|
| **1. Module** | Organization (L5) |
| **2. Section** | Demo seed data |
| **3. Reason** | AriaSanatDemoOrgSeeder may still be stubbed; restore from a206128c only if rich demo needed. |
| **4. Layer type** | Backend |
| **5. Created at** | 2026-09-29; registered 2026-10-06T09:53:00+02:00 |
| **6. Owner decision** | ثبت اولیه — optional |
| **7. Suggestion** | Only if sales demos require multi-entity showcase; otherwise leave stub. |
| **Status** | DEFERRED |

---

## C. Cross-cutting / Platform blockers (index only)

| Debt ID | Title | Blocks | Status |
|---------|-------|--------|--------|
| DEBT-PLT-001 | SaaS Admin feature catalog + pack purchase API | DEBT-ID-009, DEBT-ORG-001/002/003/008 | **PARTIAL/CLOSED BE** — L1 FeatureCatalogService + APIs + outbox live (ADR-SAASADM-001). Admin UX shell still open (SAASADM-P2/P7). |
| DEBT-PLT-002 | System settings page (move identity-only dual-approval toggle) | DEBT-ID-002 residual | OPEN — SAASADM-P3 |
| DEBT-PLT-003 | Tenant retention setting UI (global) | DEBT-ID-003, DEBT-ORG-007 | OPEN — SAASADM-P3 |

---

## D. Close-out checklist (Identity + Organization)

Use this when deciding to **seal L4 / L5 foundation**:

**Identity seal candidates (2-week window if prioritized):**
- [ ] DEBT-ID-001 Change-password endpoint
- [ ] DEBT-ID-002 Dual-approval UX (minimal)
- [ ] DEBT-ID-006 PermissionSeeder completeness
- [ ] DEBT-ID-007 User-roles GET (if still missing)
- [ ] DEBT-ID-005 SoftDeletes/row_version audit

**Organization seal candidates (same window):**
- [ ] DEBT-ORG-006 Ownership HTTP routes (if needed)
- [ ] DEBT-ORG-007 Retention/purge P0 for companies/branches/departments
- [ ] DEBT-ORG-009 only if load issues observed

**Explicitly NOT required to seal (blocked/deferred):**
- FE residual of packs (hub cards) — can ship after Admin shell
- Intercompany Acc/Ops, Sales consumers, SSO/SCIM, optional FE polish

---

## Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-10-06 | Initial register: remaining L4 Identity + L5 Organization debts from status report; register law locked |
| 1.1 | 2026-10-06 | SAASADM-P0/P1 BE verified: FeatureCatalog + Org gates + tests; DEBT-ORG-002 CLOSED (BE); ORG-001/008/ID-009 PARTIAL; PLT-001 PARTIAL |
