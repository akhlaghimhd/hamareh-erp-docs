# ADR-ORG-002 — Intercompany Target Architecture v1.0

- **Status:** Accepted (product direction: full competitive multi-entity IC)
- **Date:** 2026-09-28
- **Org-IC-P1:** **CLOSED** 2026-09-28 (config + UI + tests + outbox events)
- **Debt / parked backlog:** `ORG_Intercompany_Status_and_Debt_v1.0.md` (same folder)
- **Supersedes partial stance in ADR-ORG-001 §ADR-ORG-06 only for sequencing detail**
- **Owner:** Organization (config) + Accounting (posting) + Sales/Purch/Inventory (operational docs)

---

## 1. Context

Product owner requires Hamareh to match **shared capabilities of major multi-entity ERPs** (SAP S/4HANA, Oracle Fusion + FCCS/EPM, NetSuite OneWorld, Microsoft Dynamics 365 Finance/BC, Sage Intacct, Acumatica, Workday Financials, Odoo Enterprise), not a skeleton mapping screen.

ADR-ORG-001 already locked: *Full intercompany + elimination is MUST*. This ADR defines the **capability matrix**, **module ownership**, and **phased delivery** so Organization does not become an accounting engine, and Accounting does not invent parallel partner maps.

**Scope boundary:** Intercompany here means legal entities **inside one tenant (one SaaS customer group)**. Cross-tenant trading between unrelated platform customers is **out of scope** and must not be mixed into Org IC tables or this ADR.

---

## 2. Competitive baseline (must-have = present in most major ERPs)

| ID | Capability | SAP | Oracle Fusion/EPM | NetSuite | D365 | Intacct | Acumatica | Workday | Odoo | Hamareh target |
|----|------------|-----|-------------------|----------|------|---------|-----------|---------|------|----------------|
| IC-01 | Partner / affiliate mapping between legal entities | Y | Y | Y | Y | Y | Y | Y | Y | **MUST — Org** |
| IC-02 | Representing customer/vendor (or BP) per direction | Y | Y | Y | Y | Y | Y | Y | via company partner | **MUST — Org + MD BP** |
| IC-03 | Document mirror rules (SO↔PO, INV↔BILL, …) | Y | Y | Y | Y | Y | Y | Y | Y | **MUST — Org config; engines consume** |
| IC-04 | Auto-create counterparty operational documents | Y Adv | partial | Y AIM | Y | rules | Y | Y direct | Y | **MUST — Sales/Purch** |
| IC-05 | Due-to / Due-from (IC AR/AP) accounts | Y | Y | Y | Y | Y | Y | Y | Y | **MUST — Accounting** |
| IC-06 | IC journal / balancing entries | Y | Y | Y | Y | Y | Y | Y | Y | **MUST — Accounting** |
| IC-07 | Elimination at consolidation | Y | Y | Y | Y | Y | Y | Y | consol journals | **MUST — Accounting + Org hierarchy/ELIM entity** |
| IC-08 | Reconciliation of unmatched / mismatched IC | Y | Y | Y | partial | Y | Y | Y | weak | **MUST — Accounting reports** |
| IC-09 | Multi-currency IC + CTA on elim | Y | Y | Y | Y | Y | Y | Y | Y | **MUST — Accounting** |
| IC-10 | Feature / license gate for IC pack | license | license | OneWorld | license | module | edition | SKU | setting | **MUST — Platform `org.intercompany`** |

### Differentiating / advanced (target after baseline)

| ID | Capability | Notable owners | Note for Hamareh |
|----|------------|----------------|------------------|
| IC-A1 | Advanced multi-stage IC sales + stock-in-transit + value-chain monitor | SAP | High complexity; after Inventory + Sales maturity |
| IC-A2 | Group valuation / profit-in-inventory elim | SAP UPA, Oracle | Requires material ledger-class costing |
| IC-A3 | Automated intercompany netting / settlement | NetSuite, Workday, banks | Cash optimization; after IC balances exist |
| IC-A4 | Transfer pricing rate sheets | Workday, SAP, tax suites | Tax/Pricing domain; compliance-heavy |
| IC-A5 | Cross-subsidiary fulfillment from other entity warehouses | NetSuite, SAP | Inventory + ATP |
| IC-A6 | Inbox/Outbox document exchange | D365 BC | Optional if same-tenant auto-mirror is native |
| IC-A7 | On-behalf-of / indirect IC | Workday | Useful for shared services centers |
| IC-A8 | Account-restricted IC posting | Acumatica | Control policy in GL |

**Product decision:** Baseline IC-01…IC-10 are **non-negotiable** for enterprise competitiveness. Advanced IC-A* are **planned** and must not be designed out of the data model (hooks and extension points remain).

---

## 3. Module ownership (binding)

| Concern | Owner module | System of truth |
|---------|--------------|-----------------|
| Which companies may trade with which | **Organization** | `erp_intercompany_partners` |
| Mirror document type policy | **Organization** | `erp_intercompany_rules` |
| ELIMINATION / CONSOLIDATION entity kinds + hierarchy snapshot | **Organization** | `erp_companies.entity_kind`, hierarchies, consol runs |
| Customer/Vendor BP representing another company | **Master Data (BP)** + Org stores logical UUID | BP tables; Org holds refs |
| IC AR/AP accounts, journals, elim postings, reconciliation, FX CTA | **Accounting** | future fin_* IC tables + events |
| Auto SO/PO/Invoice/Bill mirror execution | **Sales / Purchasing** | consume Org rules |
| Stock transfer / fulfillment across companies | **Inventory** | consume partners + rules |
| Transfer pricing methodologies | **Pricing / Tax** (future) | rate sheets; Org only flags policy ref |
| Sellable feature flag | **Platform / SaaS Admin** | `org.intercompany` |

**Law:** No physical FK across modules. Org publishes partners/rules via API + domain events; consumers resolve by UUID.

---

## 4. Organization-phase deliverables

### Org-IC-P1 — CLOSED (2026-09-28)

1. Admin UX for partners and rules (list, create, update, soft-delete, activate/deactivate).
2. Partner fields: from/to company, optional `partner_customer_id` / `partner_vendor_id`, notes, is_active.
3. Rule fields: code, name, source/target doc types (catalog), auto_create_mirror, is_active, notes.
4. List APIs enrich with company display names.
5. Doc-type catalog endpoint.
6. Permissions: `organization.intercompany.view|manage`.
7. Outbox events: partner/rule upserted + deleted (`organization.intercompany_*.v1`).
8. FE deferred-phase hints on `/dashboard/organization/intercompany`.
9. Tests: `OrgP5SalesPurchIcTest` extended.

**Explicit non-goals for Organization code:** posting journals, creating sales/purchase documents, running elimination math, netting cash, cross-tenant network trading.

**Platform flag** `org.intercompany` enforcement remains debt IC-D-PLT-01 until SaaS Admin catalog ships.

---

## 5. Downstream contracts (must not break)

- `organization.intercompany_partner.upserted.v1` / `.deleted.v1`
- `organization.intercompany_rule.upserted.v1` / `.deleted.v1`
- Accounting consumes partner pair + elim hierarchy nodes for period close.
- Sales/Purch read `auto_create_mirror` + doc type pair before generating counterparty docs.

---

## 6. Sequencing

| Phase | Scope | Status |
|-------|--------|--------|
| **Org-IC-P1** | Config center complete | **CLOSED** |
| **Acc-IC-P1** | Due-to/Due-from, IC journal, elim postings, reconciliation | PARKED — see debt file |
| **Ops-IC-P1** | SO↔PO / Invoice↔Bill auto-mirror | PARKED — see debt file |
| **Acc-IC-P2** | Netting, multi-currency CTA hardening | PARKED |
| **Adv-IC** | Transfer pricing, stock-in-transit, multi-stage value chain | PARKED |

Detailed debt IDs: `ORG_Intercompany_Status_and_Debt_v1.0.md`.

---

## 7. Decision

Accepted: Hamareh targets **full competitive IC baseline (IC-01…IC-10)** **within a single tenant**. Organization owns configuration SoT; operational and accounting capabilities are mandatory follow-on work tracked in the debt document — not optional polish.

---

## 8. Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-09-28 | Full matrix + ownership + Org-IC-P1 scope |
| 1.0.1 | 2026-09-28 | Org-IC-P1 CLOSED; debt link; out-of-scope cross-tenant |
