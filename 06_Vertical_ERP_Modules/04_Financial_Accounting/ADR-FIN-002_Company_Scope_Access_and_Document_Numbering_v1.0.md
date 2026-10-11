# ADR-FIN-002 — Company Scope Access & Document Numbering (Module-Wide)

**Status:** ACCEPTED 2026-10-11  
**Module:** FinancialAccounting (`App\\Modules\\FinancialAccounting`)  
**Related:** ADR-ID-ORG-003 (Holding Access), ADR-FIN-001, FIN_P0_01 Ownership  
**Path:** `06_Vertical_ERP_Modules/04_Financial_Accounting/ADR-FIN-002_Company_Scope_Access_and_Document_Numbering_v1.0.md`

---

## 1. Context

Holding model (ADR-ID-ORG-003): **one tenant per group**. Isolation of legal entities is via **`company_id` + Scope**, not separate tenants.

Finance surfaces (journals, CoA, periods, bank, tax, IC, reports) must not re-decide access/numbering per screen. This ADR locks **module-wide** rules so parent / child / SME paths stay consistent and no corner remains open for later debt.

---

## 2. Decision summary (LOCKED)

| ID | Decision |
|----|----------|
| D1 | Every operational FIN row that belongs to a legal entity has **`company_id` NOT NULL** (or is reachable only through a parent that has `company_id`). |
| D2 | **Authorization** = Permission ∧ **Scope company set**. Permission alone never grants cross-company access. |
| D3 | **List/read/write** always intersect user Scope. No “all tenant companies” without Scope. |
| D4 | **Document numbers** are **per `(tenant_id, company_id, fiscal_year, series_kind)`** — never group-wide series. |
| D5 | Journal series kinds: **`DR`** (draft) and **`JE`** (posted/reversed). Format **C+A**: `{KIND}-{FY}-{SEQ6}`. |
| D6 | Fiscal year in the number comes from the **company’s** period/calendar assignment, not a global UI year. |
| D7 | SME / no company scopes → soft path = **primary company only** (unchanged product law). |
| D8 | Branch is an **analytic dimension** when present; **number series stay per company** unless a future ADR says otherwise. |
| D9 | Intercompany = **two documents, two series** (one per company) + link id; not one shared number. |
| D10 | Service-layer asserts (not only UI/Controller). |

---

## 3. Scope matrix (all FIN boxes)

Apply the same three modes everywhere:

| Mode | Condition | List default | Create `company_id` |
|------|-----------|--------------|---------------------|
| **SME** | No company Scope (or single implicit primary) | Primary only | Fixed primary |
| **Child** | Hard Scope = one company | That company only | Fixed that company |
| **Parent** | Scope = two or more companies | Filter: all allowed \| one company | Mandatory selector from allowed set |

### 3.1 Surfaces in scope of this ADR

| Surface | `company_id` owner | Number series? | Notes |
|---------|--------------------|----------------|-------|
| Journal entries + lines | Journal header | **Yes — DR/JE** | Primary focus of first implementation wave |
| Ledgers | Ledger | No doc# | List/create gated by Scope |
| Chart of accounts / accounts | CoA → company or shared policy per existing FIN law | No doc# | Account codes remain CoA rules; access still Scope |
| Fiscal period controls | Control row / company assignment | No | Open/close only for scoped companies |
| Bank accounts / recon | Bank account → company | Recon doc# if any → per company | |
| Tax reports / runs | Run header | Run id per company+period | |
| IC journal legs | Each leg’s company | Each leg uses **that** company’s JE series | Pair link separate |
| Financial reports (TB, GL, …) | Filter company(ies) ⊆ Scope | N/A | Parent may multi-select allowed companies |
| Sequences table | N/A | **Yes — infrastructure** | `fin_document_sequences` |

Any **new** FIN entity with legal-entity ownership MUST declare: (1) `company_id` column or join path, (2) Scope assert helper, (3) whether it needs a document series.

---

## 4. Access rules (coded)

| Code | Rule |
|------|------|
| **FIN-CSN-A01** | `allowedCompanyIds(user)` from JWT/Scope; empty company-scope in SME soft path → `{primaryCompanyId}`. |
| **FIN-CSN-A02** | `assertCanAccessCompany(companyId)` on every show/update/delete/post/reverse/restore. |
| **FIN-CSN-A03** | `assertCanAccessJournal(journal)` ≡ assert on `journal.company_id`. |
| **FIN-CSN-A04** | List queries: `where company_id in allowed` (+ optional filter equality). |
| **FIN-CSN-A05** | Child never sees sibling company data (403, no existence leak). |
| **FIN-CSN-A06** | Parent sees only companies in Scope (not whole tenant org tree by default). |
| **FIN-CSN-A07** | 403 messages Persian, non-technical. |
| **FIN-CSN-A08** | Bulk ops: prefer single-company filter; if multi-company selection, process per-company with per-row Scope check or disable bulk. |

---

## 5. Data isolation (coded)

| Code | Rule |
|------|------|
| **FIN-CSN-D01** | `tenant_id` + PostgreSQL RLS ENABLE+FORCE on all operational FIN tables (existing law). |
| **FIN-CSN-D02** | Journal (and other entity docs): `company_id` NOT NULL. |
| **FIN-CSN-D03** | Unique: `(tenant_id, company_id, entry_number)` where `entry_number` not null; separate unique for `draft_number` when present. |
| **FIN-CSN-D04** | IDOR guard after load by id. |
| **FIN-CSN-D05** | Soft-delete + retention purge remain global FIN law; purge still Scope/tenant safe. |
| **FIN-CSN-D06** | Cross-module refs stay logical UUID (no physical FK to Org). |

---

## 6. Document numbering (coded)

| Code | Rule |
|------|------|
| **FIN-CSN-N01** | Series key = `(tenant_id, company_id, fiscal_year, kind)` with `kind ∈ {DR, JE}` (extensible for later doc types: e.g. `BR` bank recon if needed). |
| **FIN-CSN-N02** | Format: `DR-{FY}-{SEQ6}` / `JE-{FY}-{SEQ6}` (ASCII stored; UI `toFaDigits`). |
| **FIN-CSN-N03** | FY = fiscal year label of the **document company’s** period (e.g. `1405`). |
| **FIN-CSN-N04** | Reset sequence at each new fiscal year **per company**. |
| **FIN-CSN-N05** | Allocate **DR** on draft create (atomic). |
| **FIN-CSN-N06** | Allocate **JE** only on post (atomic, gapless within JE series). |
| **FIN-CSN-N07** | Soft-deleted draft consumes DR; no reuse. |
| **FIN-CSN-N08** | After post: `entry_number` immutable; keep `draft_number` for audit/UI history. |
| **FIN-CSN-N09** | Reverse: new **JE** same company; set `reverses_entry_id`. |
| **FIN-CSN-N10** | UI: DRAFT → show DR; POSTED/REVERSED → show JE; never empty placeholder. |
| **FIN-CSN-N11** | No group-level continuous series for official journals. |
| **FIN-CSN-N12** | Copy within company → new DR; copy cross-company → new doc in target series only (or forbid). |
| **FIN-CSN-N13** | Table `fin_document_sequences` (or equivalent) with row lock / transactional nextval. |

Other document types later register a new `kind` under the same table and ADR amendment — do not invent parallel mechanisms.

---

## 7. Create / update UX (coded)

| Code | Rule |
|------|------|
| **FIN-CSN-C01** | API create body includes **required** `company_id`. |
| **FIN-CSN-C02** | UI company selector when `allowed.length > 1`; hidden/fixed when `length == 1`. |
| **FIN-CSN-C03** | Ledger, period, CoA, bank, cost center must be valid for selected company. |
| **FIN-CSN-C04** | Client form draft (localStorage) stores `company_id`; restore onto another company requires confirm or discard. |
| **FIN-CSN-C05** | Never silently substitute primary when user selected another company. |

---

## 8. List / report UX (coded)

| Code | Rule |
|------|------|
| **FIN-CSN-L01** | Optional `company_id` query param; omit = all **allowed**. |
| **FIN-CSN-L02** | Company filter control only if `allowed.length > 1`. |
| **FIN-CSN-L03** | Company column when multi-company results possible. |
| **FIN-CSN-L04** | Status counters respect active company filter. |
| **FIN-CSN-L05** | Reports: same Scope intersection; parent may pass multiple allowed ids. |

---

## 9. Out of scope (explicit)

- Per-**branch** official journal series (default denied).
- Group-wide single JE sequence for parent.
- Changing `entry_number` after post.
- Using consolidation reports as a substitute for operational journal ACL.
- Cross-tenant trading (already abandoned in Org IC ADR).

---

## 10. Work breakdown — phases & item codes

Implementation order is mandatory: **foundation → journals end-to-end → roll out helpers to other surfaces → tests/docs closure**.

### Phase F0 — Foundation (BE) — **do first**

| Code | Item | Level |
|------|------|-------|
| **FIN-CSN-F0-01** | Shared `FinanceCompanyScope` helper (`allowedCompanyIds`, `assertCanAccessCompany`) reusable by all FIN services | Simple |
| **FIN-CSN-F0-02** | Migration `fin_document_sequences` + model + `DocumentSequenceService::next(kind, companyId, fiscalYear)` | Simple |
| **FIN-CSN-F0-03** | Journal columns: ensure `draft_number`, keep `entry_number`; backfill strategy for existing rows | Simple |
| **FIN-CSN-F0-04** | Unique indexes for draft/entry numbers per company | Simple |
| **FIN-CSN-F0-05** | Wire Scope assert into `JournalEntryService` list/show/create/update/delete/post/reverse/restore | Simple |
| **FIN-CSN-F0-06** | Allocate DR on create, JE on post; reverse allocates JE | Medium |
| **FIN-CSN-F0-07** | API: require `company_id` on create; list filter ∩ Scope | Simple |
| **FIN-CSN-F0-08** | PHPUnit: sibling deny, parent multi, SME primary, sequence isolation, no reuse after delete | Medium |

### Phase F1 — Journals UI (FE)

| Code | Item | Level |
|------|------|-------|
| **FIN-CSN-F1-01** | Resolve `allowedCompanies` from auth/org; SME vs child vs parent modes | Medium |
| **FIN-CSN-F1-02** | List company filter + column + counters scoped | Medium |
| **FIN-CSN-F1-03** | Create/edit: company selector; stop hard-coding primary only | Medium |
| **FIN-CSN-F1-04** | Display DR/JE per N10; never empty / English draft stub | Simple |
| **FIN-CSN-F1-05** | Bulk guarded when multi-company filter | Simple |
| **FIN-CSN-F1-06** | Form localStorage includes company_id | Simple |

### Phase F2 — Roll-out same pattern to other FIN boxes

| Code | Item | Level |
|------|------|-------|
| **FIN-CSN-F2-01** | Ledgers list/create Scope | Simple |
| **FIN-CSN-F2-02** | CoA / accounts access ∩ company policy + Scope | Medium |
| **FIN-CSN-F2-03** | Period controls open/close per scoped company | Simple |
| **FIN-CSN-F2-04** | Bank accounts + recon list/create Scope (+ series if doc numbered) | Medium |
| **FIN-CSN-F2-05** | Tax report runs Scope | Simple |
| **FIN-CSN-F2-06** | Report endpoints (TB/GL/…) company filter ∩ Scope | Medium |
| **FIN-CSN-F2-07** | IC legs: each side company Scope + own JE series | Advanced |

### Phase F3 — Hardening

| Code | Item | Level |
|------|------|-------|
| **FIN-CSN-F3-01** | Concurrency test on sequences | Medium |
| **FIN-CSN-F3-02** | Audit log fields unchanged; numbers visible in history | Simple |
| **FIN-CSN-F3-03** | PermissionSeeder labels FA unchanged; document any new perms only if needed | Simple |
| **FIN-CSN-F3-04** | Update FIN_Technical_Work_Breakdown + capability catalog reference to this ADR | Simple |

---

## 11. Acceptance (module)

- [ ] Parent with Scope {A,B} lists journals of A and B only; filter works.
- [ ] Child Scope {B} cannot GET journal of A (403).
- [ ] Create on B yields `DR-1405-0000N` from B’s sequence; post yields `JE-…` on B.
- [ ] A and B may both have `JE-1405-00001`.
- [ ] SME soft path unchanged (primary only).
- [ ] Same Scope helper used by at least journals + one other FIN service (F2 start).
- [ ] No UI path creates a journal under primary while user believes they selected another company.

---

## 12. Reference for future chats

Cite: **`ADR-FIN-002`** or item codes **`FIN-CSN-*`**.  
Implementation default branch: **`develop`** (BE + Front).  
Docs: this file on **`hamareh-erp-docs` main**.

**Numbering format locked unless superseded by a new ADR:** C+A dual series DR/JE, yearly reset per company.
