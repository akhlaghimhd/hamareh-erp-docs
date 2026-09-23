# ADR-ORG-001 — Organization Layer Architecture Decisions v1.0

- **ADR ID:** ADR-ORG-001
- **Date:** 2026-09-23
- **Status:** Accepted
- **Context:** Completion of Organization / Legal Entity layer to enterprise ERP parity
- **Related roadmap:** `ORG_Layer_Completion_Roadmap_v1.0.md`

---

## Decision drivers

Product direction requires **full enterprise multi-entity support**. Decisions prefer maximum capability that remains compatible with:

- Existing `erp_companies`, `erp_branches`, `erp_departments`
- Tenant RLS and Scope (`COMPANY`, `BRANCH`, `DEPARTMENT`)
- Modular monolith boundaries (no cross-module physical FKs)

---

## ADR-ORG-01 — Legal entity modeling

**Decision:** Single table `erp_companies` remains the Legal Entity store. Add `entity_kind`:

- `OPERATING` — normal legal entity
- `CONSOLIDATION` — grouping node for reporting (optional use)
- `ELIMINATION` — elimination entity for intercompany eliminations

**Rejected:** Immediate split into SAP-style Company + Company Code tables (would break current identifiers and Scope).

**Consequences:** Consolidation roles are data-driven; Accounting integrates via `company_id` + kind rules.

---

## ADR-ORG-02 — Branch vs Plant

**Decision:** Extend `erp_branches` to Plant-grade operational sites (`branch_kind`, logistics flags, optional `parent_branch_id`, logical warehouse mapping).

**Rejected:** New parallel `erp_plants` table in phase-0 (duplicate hierarchy risk).

**Consequences:** Inventory keeps warehouses; Organization owns site master; link is logical UUID.

---

## ADR-ORG-03 — Business Unit

**Decision:** Introduce `erp_business_units` and assignment to one or more companies.

**Consequences:** Management reporting dimension independent of pure legal tree; future Scope type `BUSINESS_UNIT`.

---

## ADR-ORG-04 — Hierarchies

**Decision:** Transactional structure remains Company → Branch → Department. Additionally, `erp_org_hierarchies` + nodes support multiple purposes (LEGAL, MANAGEMENT, TAX, ESTABLISHMENT, CUSTOM).

**Consequences:** Policies and reports can use purpose-specific trees without rewriting transactional FKs.

---

## ADR-ORG-05 — Identity richness

**Decision:** Internal Legal Entity attributes live on `erp_companies` (+ polymorphic address/contact). External customers/suppliers remain Business Partner aggregate.

**Consequences:** Controlled overlap of field names is acceptable; domains stay separate.

---

## ADR-ORG-06 — Intercompany

**Decision:** Intercompany is MUST. Deliver in layers: mapping & rules (Org) → posting/elimination execution (Accounting integration events).

**Consequences:** Org owns partner map and elimination entity rules; Fin owns journals.

---

## Compliance with platform laws

- Law 1.x tenancy & soft delete: enforced on all new tables
- Law 2.x module boundaries: logical references only across modules
- Law 4.x Scope chain: extend, do not replace
- Law 5.x master data ownership: Organization owns companies/branches/departments/BU/cost centers as listed in roadmap

---

## Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-09-23 | Initial acceptance with full-enterprise roadmap |
