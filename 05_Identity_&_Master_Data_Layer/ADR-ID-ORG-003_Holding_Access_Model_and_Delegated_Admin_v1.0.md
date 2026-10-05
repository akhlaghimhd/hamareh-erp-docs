# ADR-ID-ORG-003 — Holding Access Model & Delegated Administration v1.0

- **ADR ID:** ADR-ID-ORG-003
- **Date:** 2026-10-02
- **Status:** Accepted / LOCKED (amended 2026-10-05)
- **Modules:** IdentityCore (Layer 4) + Organization (Layer 5)
- **Related:** ADR-ORG-001, ORG_Smart_Hierarchy_Product_Law_v1.0, Law 4.x (User → Role → Permission → Scope → Resource), ORG_DDL_Notes §9 feature packs
- **Companion product law:** same document § Product Law (binding)

---

## 1. Context

Holding groups need:

1. **Parent (holding) company** operators who can see the whole group — either flat across all companies or filtered per company — without losing their own company’s data.
2. **Child company** operators (e.g. HR admin of Company A) who act as **admin of that company only**, with no visibility or control over sibling companies.
3. **Single-company tenants** (no holding) that keep the simple one-level path with no holding UX burden.

Today the platform already supports multi-company structure and resource Scopes inside **one tenant**. Identity administration (members, roles, role assignment) is still largely **tenant-wide**. This ADR locks the product and architecture rules so Scope applies to **both operational data and Identity administration** without splitting a holding into multiple tenants.

---

## 2. Decision summary

| Decision | Choice |
|----------|--------|
| Tenancy for a holding group | **Single tenant** for the entire legal group |
| Isolation between companies inside the tenant | **Scope** (`COMPANY` / `BRANCH` / …), not a second tenant |
| Company tree shape | **Single root** per tenant (primary / platform-provisioned company); no forest of parent-less companies |
| Parent company access | **Group-wide** (effective all companies in tenant) with UI choice: **flat aggregate** vs **per-company filter**; parent’s own company data always retained; **minimum view** of all descendant data/users/documents |
| Child company access | **Hard-scoped** to own company (and its branches); no sibling visibility; no cross-company admin; access to parent data only if parent explicitly allows |
| Single-company tenant | Unchanged simple path; no holding UX |
| Identity catalog (roles / permissions definitions) | Remains **tenant-level SoT** (Core Only Once) |
| Identity **administration** of users & role assignments | **Scope-aware Delegated Admin** — child-company admins only manage memberships and assignments inside their Scope |
| Alternative rejected | One tenant per legal company (breaks IC, consolidation, single sign-on membership, shared role catalog) |
| Alternative rejected | Multiple root companies per tenant (enables pseudo multi-tenant under one subscription; breaks clear group ownership) |

---

## 3. Product Law (binding)

### 3.1 Actor classes

| Actor class | Typical assignment | Effective Scope |
|-------------|-------------------|-----------------|
| **Tenant Owner / Group admin** | `is_owner` or explicit group-wide scopes | All companies in tenant (or explicit multi-company set) |
| **Parent / Holding operator** | Scopes covering all (or selected) companies in the group | Group-wide; may switch view mode |
| **Child company admin** (e.g. HR of Co. A) | `COMPANY` scope = Company A (+ optional branch scopes under A) | Only Company A resources and only Identity subjects in A |
| **Operational user** | Narrower role + scope | Data only; no Identity admin |
| **Single-company tenant user** | Default / primary company scope | Same as today; no multi-entity picker required |

### 3.2 Single-root company tree (LOCKED 2026-10-05)

Aligned with NetSuite OneWorld (one root subsidiary) and ADR-ORG-001 primary company at onboarding.

1. **Exactly one root** per tenant: the **primary company** provisioned by the **Platform Owner** at tenant onboarding (`is_primary = true`, `parent_company_id = null`).
2. **Second and later companies** (requires `multi_company` pack) **MUST** have a non-null `parent_company_id` pointing to an existing company in the same tenant. Creating a second parent-less company is **forbidden** for tenant users.
3. Tenant users **MUST NOT**:
   - clear `parent_company_id` of any non-root in a way that creates a second root;
   - re-parent the primary/root to hang under another company;
   - transfer `is_primary` away from the platform-provisioned root without Platform Owner authority.
4. **Platform Owner only** may perform exceptional root re-anchoring (support tooling / audited platform admin path). Customer-facing UI and tenant APIs never expose this.
5. `ELIMINATION` (and typically `CONSOLIDATION`) entities always require a parent (already enforced for ELIMINATION).

Rationale: prevents using one subscription as multiple unrelated businesses; keeps clear legal/group ownership for consolidation, IC, and Scope delegation.

### 3.3 Parent (holding) rules

1. Parent may **select view mode** on lists and reports:
   - **Flat / group** — all companies in one result set (with company dimension column where relevant).
   - **Per-company** — filter to one company at a time.
2. Parent’s **own company data must never disappear** when switching modes; primary/parent company remains first-class.
3. **Visibility asymmetry (LOCKED 2026-10-05):**
   - **Parent → children:** parent (group-wide / owner path) always retains **at least view** of descendant companies’ data, documents, and users; filtering between children is allowed; management of parent’s own company remains normal admin.
   - **Children → parent:** child-scoped actors **do not** see parent (or sibling) data unless the parent **explicitly grants** Scope/permission.
   - Data of a parent company must **not** be hideable from legitimate group-wide parent operators for ordinary tenant configuration (no customer toggle that blinds the holding from subsidiaries).
4. Parent with group-wide scope may perform Identity admin across the group (subject to permissions and any future approval policy).
5. UI must not force hierarchy-tree expertise for day-to-day filters; company picker + flat toggle is enough for v1.

### 3.4 Child company rules

1. Child admin **sees and manages only** resources whose company (or branch under that company) is inside their effective Scope.
2. **Delegated Identity admin:** list/create/update members, assign/revoke roles, and related Identity UI/API **must be filtered and enforced** to subjects whose effective company scope is within the admin’s Scope.
3. Sibling companies are **invisible** for operational and admin surfaces (no leak via search, export, or report defaults).
4. Child cannot elevate self to group-wide scope; only group admin / owner (or platform-defined grant path) may expand scopes.

### 3.5 Single-company (non-holding) rules

1. No obligation to show multi-company pickers or holding view modes.
2. Primary company + optional hidden default HQ branch remain the implicit path (ORG_DDL_Notes §9).
3. Same Scope machinery may exist internally; product UX stays simple.

### 3.6 Catalog vs assignment

1. **Role and permission definitions** stay tenant-wide (`tenant_roles`, `tenant_permissions`) — one Core catalog.
2. **Assignments** (`tenant_user_roles`, `tenant_user_scopes`) are what Delegated Admin constrains.
3. Optional future: company-labeled role *templates* or naming conventions — not separate RBAC engines per company.

### 3.7 Enforcement (non-negotiable)

1. Scope limits must be enforced in **backend services and queries**, not only FE hide/show.
2. Identity admin endpoints (members, role assign/revoke, scope assign) MUST call the same Scope assertion path used for resources (`assertCurrentUserHasAccessTo` / RequireScope family or successor).
3. RLS remains tenant-level; company isolation is **application Scope** on top of tenant RLS (Law 1.x unchanged).
4. Hierarchy trees (LEGAL/ESTABLISHMENT/…) are **not** a substitute for Scope (Smart Hierarchy Product Law §7 — Scope remains independent).
5. Single-root rules (§3.2) enforced in **CompanyService** (and equivalent) on create/update; FE mirrors for UX only.

---

## 4. Architecture consequences

### 4.1 IdentityCore

- Extend Scope evaluation so **management of TenantUser / role assignment** is subject to the actor’s company (and branch) scopes.
- Define clear rules for users with **multiple company scopes** (union of allowed companies).
- Owner / group-wide scope bypass remains explicit and auditable.
- Events and membership history must record actor and effective scope context where relevant.
- Parent visibility of children is the default for group-wide/owner; never implement a tenant setting that hides all child operational data from the holding operator path.

### 4.2 Organization

- Parent/child legal relations continue via `parent_company_id` / ownership; they inform hierarchy and reporting, not a second tenancy model.
- Feature packs `org.multi_company` / `org.multi_branch` still gate structure UX; this ADR does not replace pack law.
- Reports and list APIs that support group view must accept an explicit company filter or “all in scope” mode consistent with §3.3.
- Create/update company APIs reject second root and protect primary root identity for tenant callers.

### 4.3 Cross-module consumers

- Any module listing tenant data by company MUST respect the caller’s Scope (existing Law 4.2–4.3).
- No module may invent a parallel “company admin” permission system outside IdentityCore.

### 4.4 Explicit non-goals (v1 of this ADR)

- Separate tenant per child company
- Separate role catalog tables per company
- Making org hierarchy the sole authorization source
- Automatic inheritance of admin rights down the legal tree without explicit Scope assignment (group-wide is explicit owner/scope grant, not silent hierarchy magic for every role)
- Cross-tenant holding (out of scope; abandoned for IC trading similarly)
- Customer-driven multi-root forests

---

## 5. Implementation phases (recommended)

| Phase | Scope | Done when |
|-------|--------|-----------|
| **H0 — Law lock** | This ADR accepted | Agents and modules cite ADR-ID-ORG-003 |
| **H1 — Scope on Identity read** | Members list/show filtered by actor company scopes | Child admin cannot list sibling members via API |
| **H2 — Scope on Identity write** | Create/update member, assign/revoke role enforced | Child admin cannot mutate sibling assignments |
| **H3 — Group view modes** | Parent lists/reports: flat vs per-company filter | Parent can switch without losing own-company data |
| **H4 — UX honesty** | FE pickers and empty states match actor class | Single-company tenants see no holding chrome |
| **H5 — Tests** | Isolation tests: sibling deny; parent allow; owner allow | PHPUnit coverage green under Docker |
| **H6 — Single-root enforcement** | CompanyService + FE: parent required after first company; primary root immutable for tenant | No second root via tenant API; tests green |

Order rule: **H1–H2 before relying on FE-only filters.** H6 may ship in parallel with structural Org work.

---

## 6. Compliance with platform laws

| Law | How this ADR complies |
|-----|------------------------|
| 1.x Tenant isolation | One tenant per customer group; RLS unchanged |
| 2.x Module boundaries | Logical refs only; Identity owns authz; Org owns structure |
| 4.1 Core Only Once | No per-company Identity engine |
| 4.2 Chain | User → Role → Permission → **Scope** → Resource still mandatory |
| 4.3 Single Scope system | Extend COMPANY/BRANCH use; do not fork Scope |
| 5.x Master data ownership | Companies remain Organization SoT |
| Configuration before customization | Behaviour driven by Scope assignments + multi_company pack, not custom code per holding |

---

## 7. Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-10-02 | Initial lock: single-tenant holding, parent flat/per-company view, child delegated admin, single-company simple path |
| 1.0.1 | 2026-10-05 | Amend: single-root tree (platform-provisioned primary); parent always min-view of children; children need explicit grant for parent data; Platform Owner only for root re-anchor |

**LOCKED.** Structural changes require v1.1+ and explicit product approval.
