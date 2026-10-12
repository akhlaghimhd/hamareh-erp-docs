# ADR-FIN-002 — Implementation Status (FIN-CSN)

**Date:** 2026-10-12  
**Code branches:** `hamarehSaasErp` / `hamarehSaasErp-Front` → `develop`  
**ADR:** `ADR-FIN-002_Company_Scope_Access_and_Document_Numbering_v1.0.md`

---

## Phase status

| Phase | Status | Notes |
|-------|--------|-------|
| **F0** Foundation BE | **DONE** | `FinanceCompanyScope`, sequences, DR/JE, JournalEntryService Scope |
| **F1** Journals FE | **DONE** | Multi-company, `displayJournalNumber` DR-05/JE-05 |
| **F2** Other FIN | **DONE (service + HTTP)** | Services scoped; controllers require `company_id` and delegate to services (Journal, Report, Period, Bank, Tax, Treasury, IC, …). **CoA** remains tenant-level (no `company_id`) — product law, not a gap in Scope helper |
| **F3** Hardening | **DONE (core)** | D7 SME primary-only; D6 FY from `fin_fiscal_periods` if present else doc date; F3-01 sequential sequence test; holding matrix test |

---

## Decision compliance

| ID | Status |
|----|--------|
| D2–D5, D8–D10 | Done |
| **D6** | Done with fallback (period table → else document date Jalali year) |
| **D7** | Done: non-owner + empty COMPANY scopes → primary only; `is_owner` → group-wide |

---

## Acceptance (ADR §11)

| Criterion | Status |
|-----------|--------|
| Parent Scope {A,B} lists only A+B | Covered at service + tests |
| Child Scope {B} cannot access A | `FinanceCompanyScopeHoldingTest` |
| DR/JE per company; A and B can share same sequence text | `JournalCompanyScopeNumberingTest` |
| SME soft path primary only | `sme_without_company_scope_only_primary_company` |
| Shared helper journals + other services | Yes |
| UI company selection (journals) | F1 done |

---

## Explicit residuals (not blockers for ADR close)

1. **CoA per-company** — only if product changes F2-02.
2. **True multi-process race** on sequences — sequential + `lockForUpdate` covered; OS-level parallel not in PHPUnit.
3. **FE company filter** on every non-journal report page — follow same journals pattern when those UIs are touched.

---

## Key tests (develop)

- `JournalCompanyScopeNumberingTest`
- `FinanceCompanyScopeHoldingTest`
- `DocumentSequenceConcurrencyTest`
- `LedgerServiceScopeTest`

Cite: **ADR-FIN-002** / **FIN-CSN-*** / this file.
