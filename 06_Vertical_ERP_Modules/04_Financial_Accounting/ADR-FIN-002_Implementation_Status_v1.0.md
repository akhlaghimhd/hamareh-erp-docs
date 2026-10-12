# ADR-FIN-002 — Implementation Status (FIN-CSN)

**Status:** **CLOSED** for coded ADR scope (2026-10-12)  
**Code branches:** `hamarehSaasErp` / `hamarehSaasErp-Front` → `develop`  
**ADR:** `ADR-FIN-002_Company_Scope_Access_and_Document_Numbering_v1.0.md`  
**Debt:** `DEBT-FIN-CSN-001` DEFERRED (CoA); `002`/`003` CLOSED

---

## Phase status

| Phase | Status |
|-------|--------|
| **F0** Foundation BE | **CLOSED** |
| **F1** Journals FE | **CLOSED** |
| **F2** Module services + HTTP | **CLOSED** (CoA tenant-level by product) |
| **F3** Hardening | **CLOSED** (D6/D7 + tests + FE report company helper) |

---

## Decisions

| ID | Status |
|----|--------|
| D2–D5, D8–D10 | Done |
| D6 FY | Period table if present, else document date |
| D7 SME | Primary company only when no COMPANY scopes |

---

## FE helper

`src/modules/finance/hooks/use-finance-company-scope.ts`  
Used by: reports, tax-reports, period-close (journals already had multi-company UX).

---

## Tests

- `JournalCompanyScopeNumberingTest`
- `FinanceCompanyScopeHoldingTest`
- `DocumentSequenceConcurrencyTest`
- `LedgerServiceScopeTest`

Cite: **ADR-FIN-002** / **FIN-CSN-*** / this file.
