# FIN Technical Work Breakdown v1.0

- **Status:** Accepted baseline for implementation tracking
- **Date:** 2026-10-07
- **Parent:** ADR-FIN-001 + FIN_Capability_Catalog_and_Wave_Plan_v1.0
- **Code module (target):** `App\Modules\FinancialAccounting`
- **Default branch:** `develop` (hamarehSaasErp / Front)

Each task has: **ID | Title | Type | Depends on | Acceptance**

Types: `Arch` `DB` `BE` `FE` `Seed` `Test` `Docs` `Integration`

---

## FIN-P0 — General Ledger foundation

| ID | Title | Type | Depends | Acceptance |
|----|-------|------|---------|------------|
| FIN-P0-01 | Align ADD-07 + table defs with Org company/fiscal SoT (no duplicate fiscal master) | Arch/Docs | ADR-FIN-001 | Decision note: which fiscal tables Finance owns vs consumes |
| FIN-P0-02 | Module skeleton `App\Modules\FinancialAccounting` (Domain/Application/Infrastructure/API) | BE | — | Autoload + empty service/controller namespaces |
| FIN-P0-03 | Migration idempotent: `fin_acc_ledgers` (leading ledger per company/tenant) | DB | P0-01 | migrate green; RLS ENABLE+FORCE |
| FIN-P0-04 | Migration: `fin_acc_accounts` CoA tree (code, type, normal_balance, is_control, parent, company_scope policy) | DB | P0-03 | unique (tenant_id, account_code) soft-delete aware |
| FIN-P0-05 | Migration: fiscal year/period **binding** (own tables only if Org has no SoT; else `fin_acc_period_controls` on logical fiscal_period_id) | DB | P0-01 | states OPEN / SOFT_CLOSED / HARD_CLOSED |
| FIN-P0-06 | Migration: `fin_acc_journal_entries` + `fin_acc_journal_items` (ledger_id, company_id, period_id, amounts NUMERIC(20,4), multi-currency columns, nullable cost_center_id/business_unit_id) | DB | P0-03..05 | CHECK debit XOR credit; FK only inside Finance BC |
| FIN-P0-07 | Migration: `fin_acc_document_sequences` (per tenant+company+fiscal_year) | DB | P0-06 | gap-safe next number under transaction |
| FIN-P0-08 | RLS policies + app_user tests pattern (no BYPASSRLS) | DB/Test | P0-04..06 | isolation test green |
| FIN-P0-09 | Models + repositories for Ledger, Account, PeriodControl, Journal | BE | P0-06 | Eloquent + soft deletes where required |
| FIN-P0-10 | `ChartOfAccountsService`: list tree / create / update / soft-delete (leaf rules) | BE | P0-09 | cannot delete account with posted lines |
| FIN-P0-11 | `FiscalPeriodControlService`: open / softClose / hardClose + post guards | BE | P0-09 | HARD_CLOSED blocks post/reverse except controlled unlock |
| FIN-P0-12 | `JournalEntryService`: createDraft / updateDraft / deleteDraft | BE | P0-09,10,11 | unbalanced draft allowed; post not |
| FIN-P0-13 | `JournalEntryService::post` — balance validate, period open, assign entry_number, status POSTED | BE | P0-12,07 | balanced; immutable after post |
| FIN-P0-14 | `JournalEntryService::reverse` — reversing entry link, audit fields | BE | P0-13 | original stays; reverse POSTED |
| FIN-P0-15 | Reporting: TrialBalanceService, ProfitAndLossService, BalanceSheetService | BE | P0-13 | filter company+period; amounts match posted lines |
| FIN-P0-16 | API routes versioned under `/api/v1/finance/...` + FormRequests | BE | P0-10..15 | 401/403 without auth/permission |
| FIN-P0-17 | Permissions: `finance.coa.*`, `finance.journal.*`, `finance.period.*`, `finance.report.view` | Seed | P0-16 | PermissionSeeder + tenant-admin grant |
| FIN-P0-18 | Seed: demo CoA (Iran minimal) + periods for demo tenant | Seed | P0-04,05 | seeder idempotent |
| FIN-P0-19 | Feature tests: CoA CRUD, post/reverse, period block, TB | Test | P0-13..15 | `#[Test]` attributes; docker compose exec app |
| FIN-P0-20 | FE: finance hub route + CoA page | FE | P0-16 | FA UI labels |
| FIN-P0-21 | FE: journal list + draft editor + post/reverse | FE | P0-16 | no full-rewrite unrelated pages |
| FIN-P0-22 | FE: period status + TB/P&L/BS screens | FE | P0-15,16 | drill-down journal id via focus pattern if required by FE URL law |

**P0 exit:** multi-company journals, period lock, reverse, TB drill, tests green.

---

## FIN-P1 — Treasury + AR + AP

| ID | Title | Type | Depends | Acceptance |
|----|-------|------|---------|------------|
| FIN-P1-01 | ADR note: bank account SoT (Org `erp_company_bank_accounts` vs Finance extension) | Arch | P0 | Law 5.1 documented |
| FIN-P1-02 | Migration: `fin_acc_cash_accounts` / use Org bank + `fin_acc_treasury_documents` (receipt/payment) | DB | P1-01 | RLS |
| FIN-P1-03 | Migration: cheque register + status enum | DB | P1-02 | lifecycle states |
| FIN-P1-04 | Migration: `fin_acc_open_items` (AR/AP) + `fin_acc_open_item_allocations` | DB | P0-06 | partial settle |
| FIN-P1-05 | Migration: bank statement + reconciliation lines | DB | P1-02 | open/reconciled |
| FIN-P1-06 | `TreasuryService` receipt/payment → optional GL draft/post policy | BE | P1-02, P0-12 | GL lines correct |
| FIN-P1-07 | `ChequeService` status transitions | BE | P1-03 | invalid transition 422 |
| FIN-P1-08 | `ArOpenItemService` / `ApOpenItemService` + aging query | BE | P1-04 | aging buckets |
| FIN-P1-09 | Manual AR/AP invoice register (until Sales/Purch full) posting to open item + GL | BE | P1-08, P0-13 | |
| FIN-P1-10 | `BankReconciliationService` | BE | P1-05 | match lines |
| FIN-P1-11 | APIs + permissions `finance.treasury.*`, `finance.ar.*`, `finance.ap.*` | BE/Seed | P1-06..10 | |
| FIN-P1-12 | Tests treasury/AR/AP/aging/recon | Test | P1-11 | green |
| FIN-P1-13 | FE treasury + cheque + recon screens | FE | P1-11 | |
| FIN-P1-14 | FE AR/AP registers + aging | FE | P1-11 | |

**P1 exit:** manual invoice → open item → receipt/payment → GL; aging works.

---

## FIN-P2 — Iran tax + Moodian + K3 alerts

| ID | Title | Type | Depends | Acceptance |
|----|-------|------|---------|------------|
| FIN-P2-01 | Tax config model: rates/exemptions per period (no hardcoded %) | DB/BE | P0 | rate change by year works |
| FIN-P2-02 | Migration: `fin_acc_tax_transactions` + Moodian submission log | DB | P2-01 | |
| FIN-P2-03 | `VatCalculationService` split net/tax on documents | BE | P2-01 | |
| FIN-P2-04 | `MoodianGateway` interface + null/mock adapter | BE | P2-02 | swappable impl |
| FIN-P2-05 | `MoodianSubmissionService` submit/status/poll | BE | P2-04 | log statuses |
| FIN-P2-06 | Ledger vs Moodian reconciliation report | BE | P2-05, P0-15 | |
| FIN-P2-07 | VAT period summary report | BE | P2-03 | |
| FIN-P2-08 | **K3** `FinanceComplianceAlertService` (missing submit, gap, rate mismatch) | BE | P2-05,06 | alerts queryable |
| FIN-P2-09 | Permissions + APIs tax/moodian/alerts | BE/Seed | P2-08 | |
| FIN-P2-10 | Tests with mocked gateway | Test | P2-09 | |
| FIN-P2-11 | FE tax settings + submission monitor + alert badges | FE | P2-09 | |

**P2 exit:** VAT split + tracked submission + actionable alerts.

---

## FIN-P3 — Integration + K1 smart drafts

| ID | Title | Type | Depends | Acceptance |
|----|-------|------|---------|------------|
| FIN-P3-01 | `fin_acc_account_determination_rules` table | DB | P0-04 | |
| FIN-P3-02 | `AccountDeterminationService` | BE | P3-01 | deterministic rules |
| FIN-P3-03 | Event consumers: sales/purchase posted → **journal draft only** | Integration/BE | P3-02, P0-12 | never auto-post default |
| FIN-P3-04 | **K1** draft payload includes per-line reason | BE | P3-03 | API returns reasons |
| FIN-P3-05 | Accept/reject draft APIs + audit who decided | BE | P3-04 | reject leaves no posted GL |
| FIN-P3-06 | Outbox: `finance.journal.posted.v1` / `reversed.v1` | BE | P0-13,14 | |
| FIN-P3-07 | Tests consumer + accept/reject | Test | P3-05 | |
| FIN-P3-08 | FE “suggested journals” inbox | FE | P3-05 | accept/edit/reject |

**P3 exit:** operational doc → reviewable draft; human confirms post.

---

## FIN-P4 — Fixed assets + dimensions + K4 close

| ID | Title | Type | Depends | Acceptance |
|----|-------|------|---------|------------|
| FIN-P4-01 | Decide cost center SoT: reuse Org `erp_cost_centers` (preferred) | Arch | Law 5.1 | DDL note supersedes duplicate if any |
| FIN-P4-02 | Migration fixed assets + depreciation entries | DB | P0-06 | |
| FIN-P4-03 | `FixedAssetService` + depreciation run → journal draft | BE | P4-02, P0-12 | |
| FIN-P4-04 | Dimension required-rules on account types | BE | P0-10, P4-01 | 422 when missing |
| FIN-P4-05 | `PeriodCloseChecklistService` blockers (drafts, recon open, depreciation pending) | BE | P1,P2,P4-03 | |
| FIN-P4-06 | **K4** guided close API (ordered steps + status) | BE | P4-05, P0-11 | |
| FIN-P4-07 | Tests assets + checklist | Test | P4-06 | |
| FIN-P4-08 | FE assets + close wizard | FE | P4-06 | |

**P4 exit:** depreciation drafts; hard close blocked by checklist.

---

## FIN-P5 — Group & intercompany

| ID | Title | Type | Depends | Acceptance |
|----|-------|------|---------|------------|
| FIN-P5-01 | Map Org IC partners/rules → GL accounts (determination) | Arch/BE | Org-IC, P3-02 | |
| FIN-P5-02 | `IntercompanyJournalService` create paired drafts | BE | P5-01, P0-12 | |
| FIN-P5-03 | Elimination journal support for ELIMINATION/CONSOLIDATION entities | BE | P5-02, Org entity_kind | |
| FIN-P5-04 | Consolidated trial balance lean | BE | P5-03, P0-15 | |
| FIN-P5-05 | Tests IC + elimination | Test | P5-04 | |
| FIN-P5-06 | FE IC workspace + consolidation view | FE | P5-04 | |

**P5 exit:** demo group IC draft + elimination path.

---

## FIN-P6 — Advanced smart K2/K5/K6

| ID | Title | Type | Depends | Acceptance |
|----|-------|------|---------|------------|
| FIN-P6-01 | Account suggestion from posting history (K2) | BE | P0-13, P3 | suggestion + reason + override |
| FIN-P6-02 | Managerial insight texts on P&L variances (K5) | BE | P0-15 | non-blocking |
| FIN-P6-03 | NL command → draft journal only (K6) | BE/FE | P0-12 | never silent post |
| FIN-P6-04 | Anomaly amount alerts on journals | BE | P0-13 | |
| FIN-P6-05 | Tests smart suggestion accept/reject audit | Test | P6-01 | |

---

## Cross-cutting (all waves)

| ID | Title | Type | Acceptance |
|----|-------|------|------------|
| FIN-X-01 | All operational tables: tenant_id, RLS, soft-delete policy per Law 1.x | DB | |
| FIN-X-02 | Posted journal: no hard delete; reverse only | BE | |
| FIN-X-03 | PHPUnit only `#[Test]` attributes | Test | |
| FIN-X-04 | docker compose exec app for artisan/test | Ops | |
| FIN-X-05 | Commits English; user reports Persian | Docs | |
| FIN-X-06 | Smart actions logged (actor, accept/reject, payload) | BE | |
| FIN-X-07 | FE URL law: no raw ids in path where platform standard applies | FE | |

---

## Implementation order (strict)

1. FIN-P0-01 → … → FIN-P0-22 (do not start P1 APIs until P0 exit met)
2. FIN-P1-*
3. FIN-P2-*
4. FIN-P3-*
5. FIN-P4-*
6. FIN-P5-*
7. FIN-P6-*

Parallelism allowed: FE of a wave after its BE APIs exist; Docs/Arch tasks first in each wave.
