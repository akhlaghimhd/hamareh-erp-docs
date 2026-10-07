# FIN-P5-01 — Intercompany Partner/Rules → GL Mapping

- **Status:** Accepted
- **Date:** 2026-10-07
- **Module:** Financial Accounting
- **Related:** ADR-FIN-001 §H; ADR-ORG-002; Org `erp_intercompany_partners` / `erp_intercompany_rules`

---

## Decision

| Concern | Owner | Finance role |
|---------|-------|--------------|
| IC partner pair (from/to company) | **Organization** | Read logical UUID |
| IC document mirror rules | **Organization** | Optional signal for Sales/Purch; Finance does not duplicate |
| Due-from / Due-to GL accounts per pair | **Finance** (`fin_acc_ic_account_maps`) | Map partner → AR/AP intercompany accounts |
| Paired IC journal drafts | **Finance** | Two DRAFT journals linked; human posts each |
| Elimination entries | **Finance** on `entity_kind=ELIMINATION` company | Draft only; reverse of IC balances |
| Consolidated TB | **Finance** | Sum POSTED of OPERATING children under CONSOLIDATION root for period |

## Rules

1. No physical FK to Org tables.
2. IC journals never auto-post (smart law).
3. Pair must exist in Org partners (`from_company_id`/`to_company_id`) when creating IC draft pair — else 422.
4. Elimination company must have `entity_kind = ELIMINATION` (read via logical company_id attributes passed by caller or Org model if same DB).
5. Consolidated TB is lean aggregation; full consol engine (currency translation, ownership %) is out of P5 scope.
