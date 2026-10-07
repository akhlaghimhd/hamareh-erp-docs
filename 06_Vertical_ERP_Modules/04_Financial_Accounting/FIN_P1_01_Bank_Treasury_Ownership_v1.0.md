# FIN-P1-01 — Bank / Treasury ownership (Law 5.1)

- **Status:** Accepted
- **Date:** 2026-10-07
- **Wave:** FIN-P1
- **Parent:** ADR-FIN-001, FIN_Technical_Work_Breakdown_v1.0, FIN_P0_01_Ownership_Decision_v1.0

## Decision

| Concern | Owner module | Table / artifact | Notes |
|---------|--------------|------------------|-------|
| Bank account **master** (IBAN, bank name, company link, active) | **Organization** | `erp_company_bank_accounts` | Already delivered in Org wave; single SoT |
| Cash / petty-cash **GL mapping** (optional) | **FinancialAccounting** | `fin_acc_cash_accounts` | Logical `bank_account_id` → Org; maps to CoA leaf for posting |
| Receipt / payment **documents** | **FinancialAccounting** | `fin_acc_treasury_documents` + lines | May create journal **draft** then post via JournalEntryService |
| Cheque **register** lifecycle | **FinancialAccounting** | `fin_acc_cheques` | Status machine; not a master of bank identity |
| AR/AP **open items** | **FinancialAccounting** | `fin_acc_open_items` + allocations | Manual register until Sales/Purch integration |
| Bank **statement** + recon | **FinancialAccounting** | `fin_acc_bank_statements` + lines | Match to treasury / journal |

## Rules

1. **No duplicate bank master in Finance.** Finance never creates a second IBAN table.
2. **Physical FK** only inside Finance BC (e.g. treasury line → treasury header). Cross-module refs are **logical UUID** (Law 2.2).
3. **Posting policy:** Treasury document may optionally produce a journal draft; default product path is human review before post (aligns with future P3 K1). P1 may call `JournalEntryService::createDraft` + explicit `post` from service when policy flag says auto-post for simple cash.
4. **RLS** on every Finance operational table: `tenant_id` + ENABLE + FORCE + nullif policy.
5. **Soft delete** on documents/registers; posted GL remains reverse-only (FIN-X-02).

## Acceptance

- [x] This note published under `06_Vertical_ERP_Modules/04_Financial_Accounting/`
- [ ] Migrations P1-02…05 follow these ownership lines
