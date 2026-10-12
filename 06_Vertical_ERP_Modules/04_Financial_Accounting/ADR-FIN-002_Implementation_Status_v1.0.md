# ADR-FIN-002 — Implementation Status (FIN-CSN)

**Status:** **CLOSED** for coded ADR scope + failure-point remediation (2026-10-12)  
**Code:** `hamarehSaasErp` / `hamarehSaasErp-Front` `develop`

---

## Failure points closed this pass

| Issue | Fix |
|-------|-----|
| `ChequeService` no Scope | `assertCanAccessCompany` on create/transition + scoped `list` + controller |
| Consol TB leaked tree companies outside Scope | `operatingDescendants ∩ allowedCompanyIds` when not group-wide |
| Cheques FE only primary | `useFinanceCompanyScope` multi-company filter/create |

Test: `ChequeCompanyScopeTest`

---

## Still product-deferred (not a silent leak)

| Item | Note |
|------|------|
| CoA per-company | DEBT-FIN-CSN-001 DEFERRED |
| Treasury / open-items / FA FE selector | BE already scoped; FE still defaults primary until same hook applied (no cross-company write without backend deny) |
| Branch-level journal series | Out of ADR scope (D8) |

---

## Phases F0–F3

| Phase | Status |
|-------|--------|
| F0–F3 core | CLOSED |
| Cheque + Consol harden | CLOSED |

Cite: **ADR-FIN-002** / **FIN-CSN-***
