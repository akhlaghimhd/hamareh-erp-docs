# ADR-FIN-002 — Implementation Status (FIN-CSN)

**Date:** 2026-10-12  
**Code branches:** `hamarehSaasErp` / `hamarehSaasErp-Front` → `develop`  
**ADR:** `ADR-FIN-002_Company_Scope_Access_and_Document_Numbering_v1.0.md`

---

## Phase status

| Phase | Status | Notes |
|-------|--------|-------|
| **F0** Foundation BE | **DONE** | `FinanceCompanyScope`, sequences, DR/JE, JournalEntryService Scope, unique series per company |
| **F1** Journals FE | **DONE** | Multi-company filter/form, `displayJournalNumber` (DR-05 / JE-05), bulk, reverse link |
| **F2** Other FIN services | **DONE (service layer)** | Ledger, Period, Bank, Tax, TB, OpenItem, FA, Consol, Treasury, IC. CoA remains **tenant-level** (no `company_id`) by existing FIN law — F2-02 N/A until product change |
| **F3** Hardening | **IN PROGRESS** | F3-01 sequence sequential test added; D7 SME primary-only; D6 FY from `fin_fiscal_periods` when table exists else doc date |

---

## Decision compliance

| ID | Status |
|----|--------|
| D2–D5, D8–D10 | Implemented |
| **D6** | Implemented with **fallback**: period table if present; else document-date Jalali year |
| **D7** | Implemented: non-owner + empty COMPANY scopes → **primary company only**; `is_owner` → group-wide; no security context → group-wide (tests/CLI) |

---

## Residual (honest)

1. **HTTP/Controller layer** for non-journal FIN surfaces may still need explicit company filter params where routes are thin.
2. **True parallel concurrency** (multi-process) not simulated in PHPUnit — sequential uniqueness under `lockForUpdate` is covered.
3. **CoA per-company** only if product changes F2-02.
4. When `fin_fiscal_periods` is fully seeded, D6 path becomes primary automatically.

---

## Key tests

- `JournalCompanyScopeNumberingTest`
- `FinanceCompanyScopeHoldingTest` (sibling deny, owner both, SME primary)
- `DocumentSequenceConcurrencyTest`
- `LedgerServiceScopeTest`

Cite: **ADR-FIN-002** / **FIN-CSN-*** / this status file.
