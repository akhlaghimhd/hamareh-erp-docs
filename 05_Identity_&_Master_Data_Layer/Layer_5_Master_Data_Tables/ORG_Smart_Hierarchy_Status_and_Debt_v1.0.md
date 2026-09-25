# ORG — Smart Hierarchy Implementation Status & Debt v1.0

**Date:** 2026-09-25  
**Related law:** `ORG_Smart_Hierarchy_Product_Law_v1.0.md` (LOCKED)  
**Code repo:** hamarehSaasErp

This file records **what was implemented** vs **explicit debt** so later work does not redo P1–P4 foundations.

---

## Implemented (do not re-implement)

| Phase | Deliverable | Location |
|-------|-------------|----------|
| P0 | Tier-aware FE + product law | Front hierarchies page; law doc |
| P1 | Auto LEGAL + ESTABLISHMENT from company/branch CRUD | `HierarchySyncService`, hooks in `CompanyService` / `BranchService`, `HierarchySyncTest` |
| P2 **minimal** | Structural ensure by data shape (company/branch/BU counts); not SaaS pack API | `ensureStructuralTrees()` |
| P3 | Health report + rebuild | `health()`, `rebuildSystemTreesForTenant()`, artisan `organization:hierarchy-health`, `organization:hierarchy-rebuild`, API `GET .../hierarchies/health`, `POST .../hierarchies/rebuild` |
| P4 | Product tree `SYS-PRODUCT` from multiple BUs (under primary company) | `syncBusinessUnit()`, hooks in `BusinessUnitService`, `HierarchySyncExtendedTest` |

### Artisan (support)

```bash
docker compose exec app php artisan organization:hierarchy-rebuild {tenant_uuid} --health
docker compose exec app php artisan organization:hierarchy-health {tenant_uuid}
```

### API (tenant JWT)

- `GET /api/v1/organization/hierarchies/health` — permission `organization.hierarchy.view`
- `POST /api/v1/organization/hierarchies/rebuild` — permission `organization.hierarchy.manage`

---

## Explicit debt (blocked or deferred — avoid duplicate design)

### D1 — P2 full: Feature-pack upgrade/downgrade (BLOCKED)

**Blocker:** SaaS Admin / Platform Owner feature catalog + per-tenant purchased packs API not productized yet.

**When unblocked:**

1. On pack `multi_company` / `multi_branch` **enabled** → call `ensureStructuralTrees()` / rebuild for that tenant.
2. On pack **disabled** → **hide/freeze** system trees in UX; **do not hard-delete** hierarchy rows.
3. Replace FE tier heuristic (counts) with pack flags as source of truth; keep counts as fallback.
4. Golden tests: upgrade empty hierarchy → trees appear; downgrade → UI hides, data remains.

**Do not** invent a second feature-flag table inside Organization module.

### D2 — P5: Report contracts (DEFERRED — no consumer modules yet)

Document per report:

| Report / capability | Hierarchy purpose | Fallback if tree missing |
|---------------------|-------------------|--------------------------|
| (TBD finance consol) | LEGAL | `parent_company_id` chain |
| (TBD logistics) | ESTABLISHMENT | direct `branch.company_id` |
| (TBD product P&L) | CUSTOM / SYS-PRODUCT | BU primary company assignment |

**Rule remains:** operational docs must not hard-depend on hierarchy before fallbacks exist.

### D3 — P6: Manual node origin (DEFERRED schema)

**Need:** optional column e.g. `node_origin` (`SYSTEM`|`MANUAL`) on `erp_org_hierarchy_nodes` (or equivalent) so point-sync does not overwrite manual edges.

Until then:

- System trees identified by codes `SYS-LEGAL`, `SYS-ESTABLISHMENT`, `SYS-PRODUCT`.
- `addNode` API may still attach nodes; rebuild may reposition system entities — document for advanced users.

**When implementing:** migration + sync policy “never change parent of MANUAL nodes” + FE badge.

### D4 — FE health badge / admin rebuild button

Optional UX on hierarchies page: show health chip + “همگام‌سازی دوباره” calling rebuild API. Not blocking backend.

### D5 — Background queue for large rebuilds

Point-sync is in-request; full rebuild for huge tenants should move to queue later (performance debt).

---

## Non-goals already decided (do not reopen without ADR)

- Full auto TAX tree as legal truth
- Hierarchy as only source for Scope
- Blocking company/branch CRUD on sync failure

---

## Resume checklist (next engineer / chat)

1. Read this file + product law v1.0.
2. If SaaS packs ready → only **D1**, wire to existing `ensureStructuralTrees` / `rebuildSystemTreesForTenant`.
3. If reports start → only **D2** contracts, no new parallel tree model.
4. If advanced manual editing required → **D3** migration first, then policy.

**Status:** Foundations P1–P4 (minimal P2) are in code; remaining work is **debt listed above**, not greenfield.
