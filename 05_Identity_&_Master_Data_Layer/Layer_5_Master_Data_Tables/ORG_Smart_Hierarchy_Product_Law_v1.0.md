# ORG — Smart Hierarchy Product Law v1.0

**Status:** LOCKED  
**Date:** 2026-09-25  
**Module:** Organization (Layer 5)  
**Related:** ORG_DDL_Notes_v1.1 (§ feature packs), ADR-ORG-001, independent packs `multi_company` / `multi_branch` / `multi_business_unit`

This document is the **single product law** for organizational hierarchies (`erp_org_hierarchies` + nodes). Implementation (backend sync, FE tiers, reports) MUST follow it. Changes require an explicit version bump (v1.1+) and ADR note.

---

## 1. Intent

Hierarchies are **smart, mostly derived maps** of entities the tenant already understands (company, branch, department, business unit, cost center). They must:

- Not require ERP modeling expertise from simple customers
- Scale by **customer tier / purchased feature packs**
- Never block day-to-day operations when a tree is missing or briefly inconsistent
- Stay **light** (point sync, not full rebuild on every click)
- Be **repairable** in one admin action (rebuild from source entities)

---

## 2. Customer tiers (how far we go)

| Tier | Detection (product) | Hierarchy UX | Tree behavior |
|------|---------------------|--------------|---------------|
| **Simple (single-line)** | No `multi_company`, no `multi_branch` (typical: 1 company + hidden default HQ branch) | Hub card **hidden** or page shows “no action needed”; no tree editing | Internal minimal tree optional; **not user-facing** |
| **Standard** | `multi_company` and/or `multi_branch` on | Read-oriented “structure view”; system-built trees; limited edit with warnings | **Auto-build & auto-sync** LEGAL + ESTABLISHMENT (and conditional others) |
| **Advanced** | Explicit advanced org / consolidation needs (or future pack); until pack exists: standard + manual node capability when multi-entity | Auto trees **plus** manual nodes labeled “دستی” | Same auto rules + validated manual nodes; platform still interprets edges |

**Rule:** Day-to-day paths (documents, lists, simple reports) use **company / branch / BU / scope** first. Hierarchy adds group rollups and advanced structure — it is **not** a hard dependency for basic ops.

---

## 3. Which trees exist and how they are born

| Purpose code | Name (FA concept) | Auto? | Source of truth |
|--------------|-------------------|-------|-----------------|
| `LEGAL` | حقوقی | **Yes** (standard+) | Companies + `parent_company_id` / ownership |
| `ESTABLISHMENT` | استقرار | **Yes** (standard+) | Company + branches (`company_id`) |
| `MANAGEMENT` | مدیریتی | Conditional | Company + high-level depts / linked BUs |
| `CUSTOM` (e.g. product lines) | سفارشی / خطوط محصول | Conditional on `multi_business_unit` + >1 BU | Business units (+ primary company assignment) |
| `TAX` | مالیاتی | **Not** fully auto in v1 | Copy-from-LEGAL wizard or manual later |

### Auto-create triggers

- Tenant gains `multi_company` or `multi_branch` (pack upgrade)
- Company created / parent changed / soft-deleted / restored
- Branch created / `company_id` changed / soft-deleted / restored
- BU created / primary company assignment changed (for product tree)
- Admin “Rebuild system trees from structure”

### Sync style (performance)

1. **Point sync** on the entity that changed (preferred)
2. **Background queue** for large moves (primary company change, pack upgrade)
3. **Full rebuild** only: pack upgrade, health=inconsistent, admin repair
4. Idempotent upsert on `(tenant_id, hierarchy_id, entity_type, entity_id)` — max one active node per entity per hierarchy
5. No cross-tenant scans; no full-tree rewrite on every list API call

---

## 4. Hard product rules (anti-crash / anti-support)

1. **Hierarchy must not sit on the critical path** of login, company/branch CRUD success, or posting operational documents. Sync failure → durable retry + health flag; user entity operation still succeeds when possible.
2. **Fallback mandatory:** If a node/tree is missing, reports/access that *can* work via direct `company_id` / `branch_id` / scope MUST do so. Empty report solely due to missing tree = product bug.
3. **System nodes vs manual nodes:** System-derived edges should be changed by editing the source entity (e.g. branch’s company), not by silent dual edit surfaces. Manual nodes (advanced) are tagged and not blindly overwritten by point sync without policy.
4. **Cycles forbidden** in auto and manual paths.
5. **Soft delete alignment:** Entity soft-delete → node inactive/hidden; restore → node returns.
6. **Primary company / hidden HQ branch:** Simple tier must not force hierarchy UX; default HQ branch stays implementation detail.
7. **BU multi-company:** In product tree, place under **primary** company assignment only (v1 law).
8. **Pack downgrade:** Do not hard-purge trees; freeze/hide. Restore pack → data still there.
9. **Admin repair:** Per-tenant “rebuild system hierarchies from companies/branches/BUs” + health status (`healthy` / `needs_sync` / `inconsistent`).
10. **No UUID in user-facing hierarchy UI** — always business name/code.
11. **Dependency registry:** Each future module must declare whether it requires a hierarchy purpose and what fallback is. Unauthorized silent dependency is forbidden.
12. **Golden tests:** pack upgrade, branch company change, primary company change, simple-tier has no user hierarchy burden, rebuild idempotency.

---

## 5. UX law by tier

### Simple
- Organization hub: hierarchy card **not emphasized / hidden** when packs off.
- Direct URL: calm message — structure is automatic; manage companies/branches only if packs allow.

### Standard
- Show trees as **system structure view**.
- Banner: trees are built from companies & branches; changing real relations is done on those forms.
- Expand nodes with **names**, not ids.
- Editing system parent in-tree either blocked or redirects with warning to source form.

### Advanced
- Same as standard, plus **add manual node** (entity picker by name).
- Warnings before high-impact changes (detach root, move large subtree).
- Manual nodes labeled; system continues validation (type, cycle, tenant).

---

## 6. Implementation roadmap (how to proceed each phase)

| Phase | Scope | Done when |
|-------|--------|-----------|
| **P0 — Law & FE honesty** | This doc locked; FE tier-aware page (hide/simplify simple; names not UUIDs; copy matches law) | User on simple path is not forced to design trees |
| **P1 — Auto LEGAL + ESTABLISHMENT** | Backend listeners/point sync on company & branch lifecycle; idempotent nodes | Creating branch creates/updates establishment node without user hierarchy UI |
| **P2 — Pack upgrade/downgrade** | On enabling multi_* packs, ensure trees exist; on disable, hide not delete | Upgrade never leaves empty hierarchy when structure data exists |
| **P3 — Health + rebuild** | Tenant hierarchy health + admin rebuild command/API | Support can fix without SQL |
| **P4 — Conditional MANAGEMENT / CUSTOM (BU)** | Only with packs + data thresholds | No tax full-auto |
| **P5 — Report contracts** | Document which reports use which purpose + fallbacks | No silent hierarchy-only reports |
| **P6 — Advanced manual** | Tagged manual nodes + non-destructive sync policy | Advanced users can extend without breaking auto |

**Order rule:** Never implement “reports require hierarchy” before P1+P2 fallbacks exist.

---

## 7. Out of scope for v1

- Full temporal versioning (`valid_from` / `valid_to` as user-facing product) — keep columns if present; product uses single active version
- Automatic TAX tree as legal truth
- Multi-parent edges in a single purpose tree
- Making Scope subsystem read only from hierarchy (Scope remains independent)

---

## 8. Acceptance checklist

- [ ] Simple tenant: no obligation to open hierarchy to operate
- [ ] Standard: LEGAL + ESTABLISHMENT reflect companies/branches after CRUD
- [ ] Sync failure does not roll back branch/company create (or user gets clear FA message + retry path)
- [ ] Rebuild is idempotent
- [ ] UI never shows raw UUIDs for nodes
- [ ] Feature packs remain independent (BU not bundled behind multi_company)

---

## 9. Document control

| Version | Note |
|---------|------|
| v1.0 | Initial lock — smart tiers, auto LEGAL/ESTABLISHMENT, fallbacks, phased roadmap |

**LOCKED.** Deviations need v1.1 + explicit product approval.
