# ADD: Layer Numbering & Code Module Mapping v1.0

- **Version:** 1.3
- **Last Updated:** 2026-09-10
- **Category:** System Blueprint & Roadmaps
- **Status:** Locked (Architectural Decision Document)
- **Project:** HamarehERP SaaS Platform

---

## 1. Purpose

This document is the **Single Source of Truth (SSOT)** for:

1. Official **layer numbers and names**
2. Mapping from **architecture layers → code modules** in `hamarehSaasErp`
3. Resolving historical naming conflicts across older blueprints and folder aliases

Any progress report, roadmap, or new design **must** use this numbering.

---

## 2. Official Layer Model (Locked)

```
Layer 1  SaaS Platform Business
        ↓
Layer 2  SaaS Admin
        ↓
Layer 3  Partner & Affiliate
        ↓
Layer 4  Identity & Access Core
        ↓
Layer 5  ERP Foundation
        ↓
Layer 6  ERP Business Modules
        ↓
Layer 7  Extensions & Integrations (future)
```

| Layer | Official name | Code module(s) | Responsibility |
|-------|---------------|----------------|----------------|
| **1** | SaaS Platform Business | `App\Modules\SaasPlatform` | Tenant, plans, subscription, platform billing, wallet, coupons |
| **2** | SaaS Admin | `App\Modules\SaasAdmin` | Platform admin IAM, audit, system settings, notifications, support |
| **3** | Partner & Affiliate | `App\Modules\PartnerLayer` (migrations under `database/migrations/partner_layer`) | Partner network, commissions, payouts |
| **4** | Identity & Access Core | `App\Modules\IdentityCore` | User, credential, profile, tenant membership, role, permission, scope |
| **5** | ERP Foundation | `Organization`, `MasterData`, `DocumentManagement` | Org structure, shared master data, documents |
| **6** | ERP Business Modules | `Accounting`, `ProcurementSales`, `Manufacturing`, `HrManagement`, `ProjectManagement`, `Inventory`, `Workflow` | Vertical bounded contexts + Workflow Engine |
| **7** | Extensions & Integrations | future | BI, mobile, external integrations |

### Layer 5 sub-modules (Foundation)

| Sub-domain | Code module |
|------------|-------------|
| Organization (Company, Branch, Department) | `App\Modules\Organization` |
| Master Data | `App\Modules\MasterData` |
| Document Management | `App\Modules\DocumentManagement` |

**Rule:** Organization is **not** part of Identity. Identity owns access; Organization owns structural entities.

### Layer 6 note – Workflow Engine

Workflow Engine (`App\Modules\Workflow`) was originally planned under Layer 5 but has been fully implemented and is owned by Layer 6 (ERP Business Modules). All workflow tables live in the tenant DB and are versioned under L6-WF-*.

---

## 3. Deprecated / Alias Names (Do Not Use for Numbering)

| Legacy phrase | Status | Replacement |
|---------------|--------|-------------|
| «SaaS Business» / folder `Layer_1_SaaS_Business` | **Retired 2026-08-29** | `Layer_1_SaaS_Platform_Business` + official name SaaS Platform Business |
| Blueprint v2.0 old: Identity = Layer 2 | **Superseded** | Identity = **Layer 4** |
| Blueprint v2.0 old: ERP Foundation = Layer 3 | **Superseded** | Foundation = **Layer 5** |
| Blueprint v2.0 old: Business modules = Layer 4 | **Superseded** | Business = **Layer 6** |
| File «ERP Core Identity Organization Layer» | **Retired 2026-08-29** | `Database Layer 4 - Identity & Access Core.md` |
| Workflow under Layer 5 (future) | **Superseded 2026-09-10** | Workflow = **Layer 6** (`App\Modules\Workflow`) |

---

## 4. Documentation Folder Alignment (current)

| Docs path | Official layer |
|-----------|----------------|
| `04_SaaS_Core_Platform_Layers/Layer_1_SaaS_Platform_Business/` | Layer 1 |
| `04_SaaS_Core_Platform_Layers/Layer_2_SaaS_Admin/` | Layer 2 |
| `04_SaaS_Core_Platform_Layers/Layer_3_Partner_Affiliate/` | Layer 3 |
| `05_Identity_&_Master_Data_Layer/Layer_4_Identity_Core/` | Layer 4 (Identity only) |
| `05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/` | Layer 5 (Master Data) |
| `06_Vertical_ERP_Modules/` | Layer 6 (including Workflow) |

Canonical DB filenames:

- Layer 1: `Database Layer 1 - SaaS Platform Business Layer.md`
- Layer 4: `Database Layer 4 - Identity & Access Core.md`

---

## 5. Database Split (Unchanged Principle)

| Logical DB | Layers |
|------------|--------|
| `hamareh_saas_core` | Layer 1, 2, 3, 4 (platform + identity) |
| `hamareh_erp_tenants` | Layer 5, 6 (foundation + verticals + Workflow) |

*(Physical deployment may start as a single PostgreSQL instance; logical ownership remains as above.)*

---

## 6. Decision Rules

1. When documents disagree on layer numbers, **this ADD wins**.
2. When documents disagree on module placement, **code module ownership + Ownership Matrix** wins for entities; layer number still follows this ADD.
3. New modules must declare: Layer number, official name, code module path, table owner.
4. Amendments require a new ADD version and update of Consolidated Blueprint + Module Architecture Map.

---

## 7. Final Statement

```
Layer 1  SaaS Platform Business     → SaasPlatform
Layer 2  SaaS Admin                 → SaasAdmin
Layer 3  Partner & Affiliate        → PartnerLayer
Layer 4  Identity & Access Core     → IdentityCore
Layer 5  ERP Foundation             → Organization + MasterData + DocumentManagement
Layer 6  ERP Business Modules       → Accounting + ProcurementSales + Manufacturing + HrManagement + ProjectManagement + Inventory + Workflow
Layer 7  Extensions & Integrations  → future
```

**Status: Locked — 2026-09-10 (v1.3 — Workflow moved to Layer 6)**
