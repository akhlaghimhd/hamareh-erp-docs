# Identity & Organization — Holding Remediation Work Breakdown v1.0

**SSOT task list for ADR-ID-ORG-004 (25 gaps)**  
**Aligned with:** ADR-FIN-003 waves where noted  
**Created:** 2026-10-14  
**Status:** Ready for development sequencing

Codes: **IDORG-W*** (waves) / **IDORG-T*** (tasks) map to gaps **G01…G25**.

---

## Wave 0 — Power boundary (P0) — START HERE

| Task | Gap | Work | Layer |
|------|-----|------|-------|
| **IDORG-T01** | G01 | Define actor kinds: Platform/Tenant technical owner, Group observer (read), Company operator (write own). Stop treating `is_owner` as operate-all. | Identity BE + docs |
| **IDORG-T02** | G02 | Multi-company tenant policy: empty COMPANY scopes ≠ group-wide; SME single-company soft path documented and tested. | Identity BE + tests |
| **IDORG-T03** | G03 | Align Identity list/mutate guards with read vs operate split (same spirit as FIN-HOLD-L03). | Identity BE |
| **IDORG-T04** | G04 | Policy: who may `createRole` / `assignPermissions`; default deny broad catalog to child company admins. | Identity BE |

**Exit:** Child company admin cannot obtain group-level power via role CRUD; owner is not silent super-operator on all companies.

---

## Wave 1 — Group observer + catalog shape (P1)

| Task | Gap | Work | Layer |
|------|-----|------|-------|
| **IDORG-T05** | G05 | Design assignable permission ceilings or role templates: Group / Company / Branch (logical layers). | Architecture + Identity |
| **IDORG-T06** | G15 | Reseed demo/default roles: group-observer, company-operator, company-user (no flat full-admin under parent for all). | Seeders |
| **IDORG-T07** | G16 | Narrow login/bootstrap grants for `is_owner` (admin pack only, not all FIN operate perms). | Auth + Seeder |
| **IDORG-T08** | G17 | Tag permissions: `Group-Read` / `Company-Operate` / `Platform-Admin` (metadata or naming convention). | PermissionSeeder + docs |
| **IDORG-T09** | G05+G02 | Permission + optional role for **group observer**; parent grants which companies (or subtree) user may **read**. | Identity BE + FE |

**Exit:** Explicit observer role/permission exists; seeds no longer imply full power for every holding user.

*Note: Overlaps FIN-HOLD-W0 — implement once, consume from FIN and Identity.*

---

## Wave 2 — Child company admin completeness (P2)

| Task | Gap | Work | Layer |
|------|-----|------|-------|
| **IDORG-T10** | G06 | Harden member list/show/update/delete under COMPANY Scope; tests for sibling + grandchild isolation. | Identity BE + tests |
| **IDORG-T11** | G07 | Role assign/revoke: only users in actor company scope; filter role picker by allowed template set. | Identity BE + FE |
| **IDORG-T12** | G08 | Role CRUD rules per actor kind (group security admin vs company admin local roles if product keeps local). | Identity BE |
| **IDORG-T13** | G10 | Scope grant/revoke: actor may only grant COMPANY/BRANCH scopes they hold; tests. | Identity BE + tests |
| **IDORG-T14** | G25 | FE members: company filter; observer read-only actions hidden; company admin UX only own company. | Front |

**Exit:** Documented delegated company admin path green under automated tests.

---

## Wave 3 — Branch as admin unit (P3)

| Task | Gap | Work | Layer |
|------|-----|------|-------|
| **IDORG-T15** | G09 | Product confirm: branch manager is Identity admin unit **or** not. If yes → HoldingAccess + member APIs honor BRANCH. | Product + BE |
| **IDORG-T16** | G14 | Nested branch + Scope evaluation; grant rules for branch scopes. | Identity + Org |
| **IDORG-T17** | G09 | FE: branch-scoped member management when enabled. | Front |

**Exit:** Written decision + implementation or explicit non-goal recorded in ADR amendment.

---

## Wave 4 — Tree, packs, IC, audit, cross-module (P4)

| Task | Gap | Work | Layer |
|------|-----|------|-------|
| **IDORG-T18** | G11 G12 | Tests: legal parent edge never auto-grants operate on descendants; observer grants explicit. | Tests |
| **IDORG-T19** | G13 G18 | **DEBT-ORG-012**: pack keys max company/branch depth; gate CompanyService/BranchService create. | Platform + Org |
| **IDORG-T20** | G19 | Audit remaining Org surfaces for feature-pack gates. | Org BE/FE |
| **IDORG-T21** | G20 | Doc + code review: no module uses hierarchy as sole auth. | Cross-cutting |
| **IDORG-T22** | G21 | Keep IC on ADR-ORG-002 rail; no free child write. | Org + FIN |
| **IDORG-T23** | G22 | Checklist: apply same read/operate split to Treasury, Sales, … when touched. | Cross-module |
| **IDORG-T24** | G23 | Dual-approval remains separate from observer grants; document interaction. | Identity |
| **IDORG-T25** | G24 | Audit events: role assign, Scope grant, access denied on Identity admin. | BE |

**Exit:** Depth packable; audit minimum; cross-module checklist published.

---

## Suggested build order (development)

```text
1) Wave 0  (T01–T04)     ← power boundary
2) Wave 1  (T05–T09)     ← observer + seeds  (shared with FIN-HOLD-W0)
3) FIN-HOLD-W1/W2        ← FinanceCompanyScope read vs operate (parallel after T01)
4) Wave 2  (T10–T14)     ← child admin polish
5) Wave 3  (T15–T17)     ← branch decision + work
6) Wave 4  (T18–T25)     ← depth pack, audit, residual
7) FIN-HOLD-W3…W4        ← branch series setting, CoA per company
```

---

## Traceability

| Gap | Tasks |
|-----|-------|
| G01 | T01 T07 |
| G02 | T02 T09 |
| G03 | T03 |
| G04 | T04 T12 |
| G05 | T05 T09 |
| G06 | T10 |
| G07 | T11 |
| G08 | T12 |
| G09 | T15 T17 |
| G10 | T13 |
| G11 | T18 |
| G12 | T18 |
| G13 | T19 |
| G14 | T16 |
| G15 | T06 |
| G16 | T07 |
| G17 | T08 |
| G18 | T19 |
| G19 | T20 |
| G20 | T21 |
| G21 | T22 |
| G22 | T23 |
| G23 | T24 |
| G24 | T25 |
| G25 | T14 |

---

## Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-10-14 | Initial breakdown from accepted 25-item list |
