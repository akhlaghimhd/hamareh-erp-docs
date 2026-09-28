# Organization Intercompany — Status and Debt v1.0

- **Related ADR:** ADR-ORG-002 Intercompany Target Architecture v1.0
- **Date:** 2026-09-28
- **Status:** Org-IC-P1 **CLOSED**; remaining phases **PARKED** until Accounting / Sales-Purch / Inventory / Platform readiness

---

## 1. Done (Org-IC-P1) — do not re-open without product change

| Item | Location |
|------|----------|
| Partner map CRUD (create/update/soft-delete/activate) | `erp_intercompany_partners`, IntercompanyService/Controller |
| Rule CRUD + document-type catalog | `erp_intercompany_rules`, catalog SO/PO/INV/BILL/… |
| List enrichment with company display names | listPartners API |
| Outbox events upsert/delete partner & rule | `organization.intercompany_*.v1` in `config/organization.php` |
| Admin UI guided page + deferred-phase hints | Front `/dashboard/organization/intercompany` |
| Feature tests | `OrgP5SalesPurchIcTest` (catalog, update, delete, reject invalid doc type) |
| Sellable flag law | `org.intercompany` in ORG_DDL_Notes §9 (enforcement still Platform) |

---

## 2. Explicitly deferred (return when prerequisites exist)

### Acc-IC-P1 (Accounting)

**Prerequisites:** GL, chart of accounts, fiscal period close, multi-currency baseline.

| Debt ID | Capability | Notes |
|---------|------------|-------|
| IC-D-ACC-01 | Due-to / Due-from (IC AR/AP) accounts | Account master + posting rules |
| IC-D-ACC-02 | Intercompany journal entries | Consume partner map |
| IC-D-ACC-03 | Elimination postings at consol | Consume ELIMINATION entity + hierarchy snapshot events |
| IC-D-ACC-04 | Reconciliation report (unmatched / amount mismatch) | After live IC balances |
| IC-D-ACC-05 | FX / CTA on elimination | After multi-currency |

### Ops-IC-P1 (Sales / Purchasing)

**Prerequisites:** SO, PO, Invoice, Bill document engines; Business Partner UUID stable.

| Debt ID | Capability | Notes |
|---------|------------|-------|
| IC-D-OPS-01 | Auto-create mirror docs when `auto_create_mirror=true` | Read Org rules + partners |
| IC-D-OPS-02 | Wire UI `partner_customer_id` / `partner_vendor_id` to BP picker | Org fields already stored |
| IC-D-OPS-03 | Paired document status / link | Optional inbox later |

### Platform

| Debt ID | Capability |
|---------|------------|
| IC-D-PLT-01 | Enforce `org.intercompany` on API + hide hub card when OFF |

### Advanced (after baseline IC-01…IC-10)

| Debt ID | Capability | Module |
|---------|------------|--------|
| IC-D-ADV-01 | Netting / settlement | Accounting + Treasury |
| IC-D-ADV-02 | Transfer pricing rate sheets | Pricing / Tax |
| IC-D-ADV-03 | Cross-company fulfillment / stock-in-transit | Inventory |
| IC-D-ADV-04 | Multi-stage value-chain monitor | Sales + Inventory |
| IC-D-ADV-05 | On-behalf-of / shared-services IC | Accounting |

---

## 3. Events already published (consumers TBD)

- `organization.intercompany_partner.upserted.v1`
- `organization.intercompany_partner.deleted.v1`
- `organization.intercompany_rule.upserted.v1`
- `organization.intercompany_rule.deleted.v1`
- (existing) `organization.elimination.requested.v1`
- (existing) `organization.consolidation.snapshotted.v1`

---

## 4. Re-entry checklist

When resuming IC work:

1. Confirm Accounting GL + period close green.
2. Start Acc-IC-P1 against this debt list (do not reinvent partner tables).
3. When Sales/Purch documents exist, implement Ops-IC-P1 mirror only via Org rules.
4. Update this file status lines; keep ADR-ORG-002 as capability law.

---

## 5. Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-09-28 | Org-IC-P1 closed; backlog parked |
