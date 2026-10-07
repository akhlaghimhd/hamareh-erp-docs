# ADR-FIN-001 — Accounting Product Decisions (Hamareh)

- **Status:** Accepted
- **Date:** 2026-10-07
- **Module:** Financial Accounting (Vertical ERP)
- **Code path (future):** `App\Modules\FinancialAccounting`
- **Related:** ADD-07 Financial Accounting Architecture; `fin_accounting_table_definitions.md`; Org (company, fiscal, IC, cost center/BU); Identity dual-approval patterns

---

## 1. Context

Hamareh needs a competitive Iranian + enterprise-grade accounting module. Product decisions were locked after:

- Benchmark of Iranian products (Sepidar, Rahkaran, Holoo, Mahak, Hesabfa, Ghiyas, …)
- Iranian legal obligations (Samaneh Moodian, VAT, performance tax basis, electronic books path)
- World ERP patterns (SAP / NetSuite / Dynamics-style GL, subledgers, period control, multi-company)
- Explicit product rule on **smart automation**: reduce repetitive work without removing user control or auditability

Existing ADD-07 remains the domain architecture baseline (event-driven posting, double-entry, RLS, polymorphic details). This ADR locks **scope, capability catalog, smart law, and delivery waves**.

---

## 2. Decision — Capability catalog (A–K)

### A) General Ledger core
Chart of accounts (multi-level), fiscal year/periods, period states (Open / SoftClosed / HardClosed), journal draft → post → reverse, balanced entry rule, sequential document numbers per company/year, no hard-delete of posted entries, trial balance / P&L / balance sheet, mandatory `company_id` (logical ref to Org).

### B) Treasury & liquidity
Cash/petty cash, bank accounts, receipts/payments (cash, card, transfer, cheque), cheque lifecycle, bank statement reconciliation.

### C) AR (customers)
Sales invoice / credit note, customer open items, partial settlement, aging.

### D) AP (suppliers)
Purchase invoice, supplier open items, partial payment, AP aging.

### E) Iran tax & compliance
Configurable VAT rates/exemptions, separate net + tax amounts, electronic invoice + Samaneh Moodian submit/status, ledger vs Moodian reconciliation, period VAT summary, performance-tax basis exports, electronic books export path.

### F) Fixed assets
Asset master, capitalization, depreciation run, disposal/sale.

### G) Analytical dimensions
Cost center / business unit on journal lines (reuse Org masters where possible); optional project later; dimensional reporting.

### H) Multi-company & group
Per-company books, intercompany postings (consume Org IC partners/rules), elimination on consolidation/elimination entities, consolidated reporting.

### I) Internal control & audit
Fine-grained permissions, dual approval for sensitive posts, full change audit, period lock, document attachments, drill-down report → journal → source.

### J) ERP integration
Posting from sales/purchase/inventory/payroll events; Org structure as dimensions/scope; single posting truth in GL.

### K) Smart automation (assistive, not autonomous)
See section 3.

---

## 3. Decision — Smart automation law

**Principle:** Smart features = **suggest + draft + explain + warn**.  
**Not:** silent final posting that hides reasoning from the user.

| Allowed | Forbidden (default path) |
|---------|---------------------------|
| Build journal **draft** from invoice/payroll/depreciation | Auto-**post** legally sensitive entries without human confirm |
| Suggest account with short reason | Force account without override |
| Alerts (unbalanced, closed period, Moodian gap, duplicate payment) | Hide compliance gaps |
| Guided period-close checklist | Close period while critical blockers remain (unless explicit override + audit) |
| Natural-language assistant that creates a **draft** | Natural-language that posts silently |

Every smart action must be: visible, rejectable/editable, and logged (who accepted/rejected).

### K capability groups
- **K1** Draft generation from operational documents / templates / depreciation
- **K2** Account suggestion (with reason; learns from prior postings later)
- **K3** Quality & compliance alerts
- **K4** Guided period close
- **K5** Managerial insights (explanations, simple cash hints — later)
- **K6** NL assistant → draft only (later)

Smart enters delivery from **P2/P3** (alerts + draft-from-invoice), not as a black-box engine in P0.

---

## 4. Decision — Delivery waves

| Wave | Focus | Catalog |
|------|--------|---------|
| **FIN-P0** | GL foundation | A + I (permissions, audit, period lock) |
| **FIN-P1** | Money & counterparties | B + C + D |
| **FIN-P2** | Iran tax | E + K3 tax/Moodian alerts |
| **FIN-P3** | Operational integration | J + K1 draft-from-invoice |
| **FIN-P4** | Corporate depth | F + G + K4 guided close |
| **FIN-P5** | Group | H + IC-assisted drafts |
| **FIN-P6** | Advanced smart | K2 mature, K5, K6 |

---

## 5. Architecture alignment notes

- Prefer **Universal Journal style**: one journal header/lines model; subledgers (AR/AP/Bank/FA) clear into GL.
- `ledger_id` and dimension columns should exist in P0 design even if only one leading ledger is active.
- Posted documents: **reverse only** (aligns Law 1.4 operational soft-delete; posted financial truth is immutable).
- No physical FK across modules; logical UUID refs (Law 2.x).
- Org already owns companies, fiscal assignments, IC partners/rules, BU/cost centers — Finance consumes, does not duplicate masters unless Finance-specific extension is required.
- ADD-07 event-driven posting from other modules remains valid for P3+.

---

## 6. Consequences

- Implementation starts at **FIN-P0** only after wave work-breakdown is tracked.
- DDL in `fin_accounting_table_definitions.md` is a starting sketch; P0 implementation must reconcile with Org fiscal/company models and this ADR (may amend table defs in a follow-up doc revision).
- Product marketing may say “smart accounting”; engineering must keep human-in-the-loop for posting and compliance.

---

## 7. References

- ADD-07_Financial_Accounting_Architecture.md
- fin_accounting_table_definitions.md
- ADR-ORG-002 Intercompany
- Project debt register (future FIN debts to be appended when implementation gaps appear)
