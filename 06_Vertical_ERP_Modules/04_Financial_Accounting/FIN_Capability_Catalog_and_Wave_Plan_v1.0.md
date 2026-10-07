# FIN Capability Catalog & Wave Plan v1.0

- **Status:** Accepted (product lock 2026-10-07)
- **Parent ADR:** ADR-FIN-001
- **Purpose:** Single checklist for product, backend, frontend, seeders, and docs progress

---

## Capability index (A–K)

| ID | Name | Wave primary |
|----|------|----------------|
| A | General Ledger core | P0 |
| B | Treasury & liquidity | P1 |
| C | AR customers | P1 |
| D | AP suppliers | P1 |
| E | Iran tax & Moodian | P2 |
| F | Fixed assets | P4 |
| G | Analytical dimensions | P4 (hooks in P0) |
| H | Multi-company & group | P5 (company_id in P0) |
| I | Control & audit | P0+ |
| J | ERP integration | P3 |
| K | Smart automation | P2–P6 |

---

## FIN-P0 — General Ledger foundation (MUST start here)

### Docs / architecture
- [ ] Confirm ADR-FIN-001 + this plan accepted
- [ ] Reconcile `fin_accounting_table_definitions.md` with Org company/fiscal models
- [ ] Permission codes list for Finance P0 in PermissionSeeder design

### Database
- [ ] `fin_acc_accounts` (CoA) + RLS + soft delete + row_version
- [ ] Fiscal year/period tables **or** formal binding to existing Org/platform fiscal entities (no duplicate SoT)
- [ ] Period state: Open / SoftClosed / HardClosed
- [ ] `fin_acc_journal_entries` + `fin_acc_journal_items`
- [ ] Leading ledger model (`ledger_id` on lines even if single ledger)
- [ ] Optional dimension columns on lines (cost_center_id / business_unit_id nullable)
- [ ] Number range / entry_number uniqueness per tenant+company+year
- [ ] CHECK: line is either debit or credit; entry balanced on post
- [ ] Indexes for TB by account/period/company

### Backend
- [ ] Module skeleton `FinancialAccounting` (Domain/Application/Infrastructure/API)
- [ ] ChartOfAccountsService CRUD + tree
- [ ] PeriodService open/soft-close/hard-close guards
- [ ] JournalService create draft / update draft / post / reverse
- [ ] Balance validation on post
- [ ] Trial balance + P&L + balance sheet query services
- [ ] Scope/company access asserts (Identity/Org patterns)
- [ ] Feature tests with PHPUnit `#[Test]`

### Seeders
- [ ] Sample Iranian CoA template (minimal legal entity set)
- [ ] Demo fiscal periods for demo tenant
- [ ] Finance permissions for tenant-admin / accountant roles

### Frontend
- [ ] Finance hub card (feature-gated if needed later)
- [ ] CoA list/tree + create/edit
- [ ] Journal list + draft editor + post/reverse actions
- [ ] Period status view
- [ ] Trial balance / P&L / BS basic screens (FA labels)

### Exit criteria P0
Two companies in one tenant can post multi-line journals; closed period blocks post; reverse works; TB drills to journal; tests green.

---

## FIN-P1 — Treasury + AR + AP

### Database
- [ ] Bank/cash accounts (or extend Org company bank accounts consumption)
- [ ] Receipt/payment documents + clearing to open items
- [ ] Cheque entities/status
- [ ] AR/AP open item tables + partial allocation
- [ ] Bank reconciliation header/lines

### Backend
- [ ] Treasury services (receipt, payment, cheque states)
- [ ] AR invoice register + settlement
- [ ] AP invoice register + settlement
- [ ] Aging reports
- [ ] Posting into GL (draft or post per policy)
- [ ] Tests

### Frontend
- [ ] Treasury screens
- [ ] Customer/supplier balance & aging
- [ ] Invoice register UIs (may wait full Sales/Purch modules — allow manual invoices in P1)

### Exit criteria P1
Receive/pay, open items, aging, and GL impact work for manual invoices.

---

## FIN-P2 — Iran tax & Moodian

### Database
- [ ] Tax rate/exemption config by period
- [ ] Tax lines on documents / `fin_acc_tax_transactions`
- [ ] Moodian submission log (status, UUID, errors)

### Backend
- [ ] VAT calculation service (rate from config, not hardcoded)
- [ ] Moodian adapter interface + submit/status
- [ ] Ledger vs Moodian reconciliation report API
- [ ] VAT period summary
- [ ] **K3 alerts:** missing Moodian submit, rate mismatch, unreconciled gaps
- [ ] Tests (including mocked Moodian gateway)

### Frontend
- [ ] Tax settings
- [ ] Submission monitor
- [ ] Reconciliation screen + alert badges

### Exit criteria P2
Sale/purchase tax split correct; submission tracked; user sees actionable compliance alerts.

---

## FIN-P3 — Integration + smart drafts (K1)

### Backend
- [ ] Consumers for sales/purchase posted events → journal **draft**
- [ ] Account determination rules table/service
- [ ] **K1** draft-from-invoice with explanation payload
- [ ] Outbox events `finance.journal.posted.v1` / `reversed.v1`

### Frontend
- [ ] “Suggested journals” inbox: accept / edit / reject
- [ ] Show reason text for each suggested line

### Exit criteria P3
Operational document can produce a reviewable draft journal; no silent post.

---

## FIN-P4 — Fixed assets + dimensions + guided close (K4)

### Database / Backend
- [ ] Fixed asset master + depreciation entries
- [ ] Enforce dimensions on selected account types
- [ ] Period close checklist engine + blockers
- [ ] **K4** guided close API

### Frontend
- [ ] Asset register + depreciation run UI
- [ ] Close checklist wizard

### Exit criteria P4
Depreciation drafts; close blocked when checklist critical items open.

---

## FIN-P5 — Group & intercompany

### Backend
- [ ] IC journal generation from Org IC rules (draft)
- [ ] Elimination entries for consolidation/elimination entities
- [ ] Consolidated TB/FS lean

### Frontend
- [ ] IC workspace + consolidation views

### Exit criteria P5
Cross-company draft + elimination path demonstrable on demo group.

---

## FIN-P6 — Advanced smart (K2/K5/K6)

- [ ] Mature account suggestion from history (K2)
- [ ] Managerial narrative insights (K5)
- [ ] NL → draft assistant (K6), still confirm-to-post
- [ ] Optional anomaly alerts on journal amounts

---

## Explicit non-goals (until later)

- Silent auto-post of tax-critical documents
- Full bank connectivity / ISO payment batches
- Full industrial cost accounting / manufacturing variance
- Replacing human accountant for statutory sign-off

---

## Current platform readiness (as of 2026-10-07)

| Area | Ready? | Note |
|------|--------|------|
| Org companies / multi-company | Yes | Finance must reference company_id |
| Fiscal assignment on company | Partial/Yes | Bind periods carefully — no second SoT |
| IC partners/rules | Yes (Org) | Consume in P5 |
| BU / cost centers | Yes (Org) | Prefer Org masters over duplicate fin_acc_cost_centers if Law 5.1 applies |
| Feature packs | Yes | Optional finance packs later |
| Identity dual-approval | Yes | Reuse pattern for sensitive post |
| Finance module code | No | Start P0 |
| Moodian connector | No | P2 |

**Important:** If Org already owns cost centers, P0/P4 should **reuse** Org cost centers (Law 5.1) and treat `fin_acc_cost_centers` in older DDL as superseded unless a Finance-only extension is justified in a DDL revision.
