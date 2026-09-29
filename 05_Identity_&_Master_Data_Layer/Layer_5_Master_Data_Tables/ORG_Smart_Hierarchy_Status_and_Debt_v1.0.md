# ORG — Smart Hierarchy Implementation Status & Debt v1.0

**Date:** 2026-09-25 (updated 2026-09-29)  
**Related law:** `ORG_Smart_Hierarchy_Product_Law_v1.0.md` (LOCKED)  
**Code repo:** hamarehSaasErp / hamarehSaasErp-Front

This file records **what was implemented** vs **explicit debt** so later work does not redo P1–P4 foundations.

---

## Implemented (do not re-implement)

| Phase | Deliverable | Location |
|-------|-------------|----------|
| P0 | Tier-aware FE + product law | Front hierarchies page; law doc |
| P1 | Auto LEGAL + ESTABLISHMENT from company/branch CRUD | `HierarchySyncService`, hooks in `CompanyService` / `BranchService`, `HierarchySyncTest` |
| P2 **minimal** | Structural ensure by data shape (company/branch/BU counts); not SaaS pack API | `ensureStructuralTrees()` |
| P3 | Health report + rebuild (API/artisan remain; **user rebuild UI removed 2026-09-28**) | `health()`, `rebuildSystemTreesForTenant()`, artisan + API |
| P4 | Product tree `SYS-PRODUCT` from multiple BUs (under primary company) | `syncBusinessUnit()`, hooks in `BusinessUnitService` |
| P4+ | Entity labels on nodes (no raw UUID in UI) | `OrgHierarchyService::enrichNodesWithEntityLabels` |
| P4+ | SYS trees read-only in FE; expand by title; auto-sync messaging | Front `hierarchies-list.tsx` (2026-09-28) |
| P4+ | CUSTOM create gated by env feature flag (until SaaS catalog) | BE `FEATURE_CUSTOM_ORG_HIERARCHY`; FE `NEXT_PUBLIC_FEATURE_CUSTOM_ORG_HIERARCHY` |
| P4+ | `assertEntityExistsInTenant` on addNode; block system-node active toggle | `OrgHierarchyService` |
| **D2** | Report contracts published | `HierarchyReportContract` + `GET .../hierarchy-report-contracts` (2026-09-29) |
| **D6** | Purpose catalog (platform table, no enum churn for labels) | `erp_hierarchy_purpose_catalog` + model + `GET .../hierarchy-purposes` (2026-09-29) |
| **D5 partial** | Async rebuild queue path | `RebuildSystemHierarchiesJob` + `POST hierarchies/rebuild?async=1` |

### Default system trees (stable set — do not grow without ADR)

| Code | Purpose | Source boxes |
|------|---------|--------------|
| `SYS-LEGAL` | LEGAL | companies + `parent_company_id` |
| `SYS-ESTABLISHMENT` | ESTABLISHMENT | companies + branches |
| `SYS-PRODUCT` | product / BU lines | primary company + business units |

Created/updated by onboarding + point-sync when org boxes change — **not** by user rebuild button.

### Artisan (support / ops only — not end-user UX)

```bash
docker compose exec app php artisan organization:hierarchy-rebuild {tenant_uuid} --health
docker compose exec app php artisan organization:hierarchy-health {tenant_uuid}
```

### API (tenant JWT)

- `GET /api/v1/organization/hierarchies/health` — permission `organization.hierarchy.view`
- `POST /api/v1/organization/hierarchies/rebuild` — permission `organization.hierarchy.manage` (ops/support; not exposed as primary FE action)
- `GET /api/v1/organization/hierarchy-purposes` — D6 catalog
- `GET /api/v1/organization/hierarchy-report-contracts` — D2 contracts

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

### D2 — P5: Report contracts — **DONE foundation 2026-09-29**

| Report / capability | Hierarchy purpose | Fallback if tree missing |
|---------------------|-------------------|--------------------------|
| finance_consolidation_group | LEGAL | `parent_company_id` chain |
| logistics_site_rollup | ESTABLISHMENT | direct `branch.company_id` |
| product_line_pnl | CUSTOM | BU primary company assignment |
| management_org_chart | MANAGEMENT | company + department + cost_center FKs |
| tax_group_view | TAX | LEGAL or parent_company_id |

Code: `HierarchyReportContract::contracts()`. Consumer modules must still wire fallbacks when they ship reports — contract is the registry.

### D3 — P6: Manual node origin (PARTIAL — schema done, policy locked)

`node_origin` (`SYSTEM`|`MANUAL`) exists; SYS trees reject manual node edits; rebuild/upsert restores system nodes. Further polish only if advanced CUSTOM editing expands.

### D4 — FE health badge / admin rebuild button (SUPERSEDED 2026-09-28)

**Product decision:** end-user rebuild button **removed**. Hierarchies page is a derived map; sync is automatic from company/branch/BU services. Health chip may remain informational only. Ops may still use artisan/API rebuild.

### D5 — Background queue for large rebuilds (PARTIAL)

Point-sync is in-request; `RebuildSystemHierarchiesJob` + `?async=1` exist. Full large-tenant performance tuning remains ops debt.

### D6 — Extensible hierarchy purpose catalog — **DONE foundation 2026-09-29**

Table `erp_hierarchy_purpose_catalog` (platform, no `tenant_id`) seeds LEGAL / ESTABLISHMENT / MANAGEMENT / TAX / CUSTOM with FA/EN labels and `allowed_entity_types`.  
API: `GET /hierarchy-purposes`.  
SYS tree codes remain fixed; user-created trees stay CUSTOM purpose under product law. Optional later: tenant-only label overrides / admin CRUD without deploy.

### D7 — CUSTOM hierarchy pack in SaaS Admin catalog (BLOCKED on D1 platform)

Today CUSTOM is gated by env:

- BE: `FEATURE_CUSTOM_ORG_HIERARCHY`
- FE: `NEXT_PUBLIC_FEATURE_CUSTOM_ORG_HIERARCHY`

**When SaaS Admin packs exist:** replace env with purchased flag (e.g. `custom_org_hierarchy`), independent of `multi_company` / `multi_branch`. Wire list/create/lifecycle UI only when pack active. Do not enable CUSTOM by default for simple tenants.

---

## Non-goals already decided (do not reopen without ADR)

- Full auto TAX tree as legal truth
- Hierarchy as only source for Scope / authorization
- Blocking company/branch CRUD on sync failure
- End-user manual rebuild as primary way to “fix” trees
- Growing SYS tree codes for every reporting scenario (use CUSTOM + catalog)

---

## Resume checklist (next engineer / chat)

1. Read this file + product law v1.0.
2. If SaaS packs ready → **D1** + **D7** (structural packs + custom hierarchy pack).
3. Reports shipping → bind to **D2** contracts + implement fallbacks in consumer module.
4. Need extra purpose **labels** → extend catalog rows, do **not** add ad-hoc SYS purposes.
5. Large tenants → tune **D5** queue.

**Status:** Foundations P1–P4 in code; product lock 2026-09-28; **D2 + D6 foundations closed 2026-09-29**. Remaining open blockers: **D1 / D7 (SaaS Admin)**.
