# ADR-SAASADM-001 — Feature Pack Model

- **Status:** Accepted
- **Date:** 2026-10-06
- **Related:** SAASADM-P0, DEBT-ORG-001/002/003/008, DEBT-ID-009, PLT-W1-01/03

## Context

Org product law requires independent sellable packs (`multi_company`, `multi_branch`, `multi_business_unit`, `custom_org_hierarchy`, `org.intercompany`). Roadmap labeled this under Layer 2 SaaS Admin, while implementation already lived in Layer 1 (`SaasPlatform`) next to plans/subscriptions.

## Decision

1. **Catalog + entitlement SoT stays in L1 (`App\\Modules\\SaasPlatform`)**  
   - Tables: `platform_feature_catalog` (platform master, no tenant_id), `tenant_feature_entitlements` (tenant-scoped + RLS).  
   - Service: `FeatureCatalogService` (`isEnabled`, `setEntitlement`, freeze/unfreeze, `assertEnabled`).

2. **L2 SaaS Admin owns admin UX/IAM** that *calls* L1 entitlement APIs — does not duplicate pack tables.

3. **Relation to L1 `plan_features`:** plan catalog remains commercial packaging; runtime gate for Org/Identity uses **entitlements** (source PLAN | ADDON | MANUAL | TRIAL). Entitlement can later be synced from subscription, but gate never reads plan tables directly from Org.

4. **Events:** `saas.tenant_feature.granted.v1` / `saas.tenant_feature.revoked.v1` on enable/disable transitions (outbox).

5. **Default:** missing entitlement = OFF (SME soft path: single company + HQ branch).

## Consequences

- SAASADM-P0 acceptance is met by L1 catalog/entitlement + APIs + outbox; P1 wires gates and Hierarchy D1 consumers.  
- Do not invent parallel pack tables in Identity or Organization.
