# ADR-FIN-003 — Holding / Company Access & Document Isolation Laws

| Field | Value |
|-------|--------|
| **Status** | **ACCEPTED / LOCKED** |
| **Date** | 2026-10-14 |
| **Module** | Financial Accounting (extends to all operational modules — Law 12) |
| **Supersedes / amends** | Amends ADR-FIN-002 where group-wide **write** was implicit for `is_owner`; numbering remains company-keyed by default with optional branch series |
| **Related** | ADR-FIN-002, ADR-ID-ORG-003, ADR-ORG-002 (IC), DEBT-ORG-012, DEBT-FIN-CSN-001 |
| **Codes** | FIN-HOLD-L01 … FIN-HOLD-L16 |

---

## 0. Product intent (owner lock 2026-10-14)

1. One tenant = whole holding group (unchanged).
2. **Holding / group observer may view and filter** financial data of subsidiaries (reports and read lists) **but must not create or edit** operational documents on a child’s books.
3. **Each company** manages **only its own** operational data (and optional branch series — see L04/L07/L10).
4. Target customers are mid-size holdings on multi-tenant SaaS; mega on-prem depth limits are deferred to Platform Owner feature packs (**DEBT-ORG-012**).
5. Assumption for implementation sequencing: full org packs purchased; depth caps later.

---

## 1. Locked laws

### FIN-HOLD-L01 — Operational write boundary = company of the actor

- Create / update / soft-delete / post / reverse of operational financial documents is allowed **only** for `company_id` values the actor is explicitly allowed to **operate** (write scope).
- Parent / holding membership alone does **not** grant write on descendants.

### FIN-HOLD-L02 — Group read via dedicated role / permission (not flat owner power)

- Cross-company **read** (lists + reports) requires an explicit **group observer** capability (new permission/role product surface).
- Parent company administrators **grant** which users may observe which companies (or subtrees).
- Read grants never imply write.

### FIN-HOLD-L03 — Read ≠ write (hard split)

- `isGroupWide()`-style **read** for observers is allowed.
- **Write** path must call a separate assert (e.g. `assertCanOperateCompany`), not the same helper that allows group read.
- Current behaviour where tenant `is_owner` can write every company is **non-compliant** and must be fixed under this ADR.

### FIN-HOLD-L04 — Primary document boundary = company; branch is secondary

- Legal / books boundary defaults to **`company_id`**.
- **`branch_id`** is a first-class dimension on documents for filter and reporting when multi-branch is in use.
- **Department / cost center / business unit** are analytical only inside the company (see L11) — not document-series boundaries.

### FIN-HOLD-L05 — Role-by-tree position

- Default posture:
  - **Group / intermediate holding node:** observe + report on allowed subtree; no write on descendant operating companies unless explicitly granted operate scope (normally not).
  - **Operating company:** full operate on self only (default).
- Exact role catalog entries are an Identity deliverable under this law.

### FIN-HOLD-L06 — `is_owner` ≠ group finance operator

- Platform/tenant **owner** is a technical/admin identity, not automatic permission to post on every subsidiary ledger.
- Group financial visibility is granted via L02 permissions, assignable by the parent company to chosen users.

### FIN-HOLD-L07 — Document numbering series

**Default mode (system out of the box)**

- Series key = `(tenant_id, company_id, fiscal_year, kind)` with `kind ∈ {DR, JE, …}`.
- Documents may still carry `branch_id` for filter/report without affecting the series key.

**Optional branch-separated mode (per company setting)**

- A company admin may enable **branch series separation**.
- Change is allowed **only effective from the start of a fiscal year** (never mid-period).
- From that effective date:
  - Document **with** `branch_id` → series `(tenant_id, company_id, branch_id, fiscal_year, kind)`.
  - Document **at company level** (`branch_id` null) → series remains `(tenant_id, company_id, fiscal_year, kind)` — **company’s own books are not removed**.
- Enabling the setting implies the company uses branches; it does **not** forbid company-level documents without branch.

### FIN-HOLD-L08 — Fiscal periods

- Period open/close is **per company**.
- Group may run **consolidated / multi-company reports** over allowed read set; group does not “own” a single forced shared close calendar unless a future product rule says so.

### FIN-HOLD-L09 — Chart of Accounts per company

- **Product decision:** CoA is **per company** (`company_id` on accounts tree / uniqueness per company).
- Amends prior “tenant-shared CoA” deferral (**DEBT-FIN-CSN-001** → implement under this ADR).
- Migration strategy for existing shared trees is an implementation work item (copy-on-write or assign company_id to primary then clone).

### FIN-HOLD-L10 — Branch on documents

- When multi-branch pack is active, UI/API should support branch on operational docs for filter and report.
- Access control for write remains **company operate scope** (L01); branch does not create a second legal entity.
- Optional series split only via L07 setting.

### FIN-HOLD-L11 — Department / BU / cost center

- Dimensions on lines / reports only.
- Must not expand write scope outside the document’s `company_id`.

### FIN-HOLD-L12 — Same law for all operational modules

- Sales, Purchasing, Inventory, Treasury, and future ops modules **must** apply the same read/write split and company operate boundary.
- Organization / Identity role seeding must be audited so roles are not accidentally “flat under parent with full write everywhere.”

### FIN-HOLD-L13 — Delegation

- **Default:** operator of company A cannot operate company B (child) without explicit grant.
- **Allowed:** explicit scope grant to operate a descendant (exception path); still audited.

### FIN-HOLD-L14 — Intercompany exception

- Cross-company operational linkage only through **formal IC** flows (Org partners/rules ADR-ORG-002; Accounting IC postings when delivered).
- IC is not a back-door for free write on arbitrary children.

### FIN-HOLD-L15 — Audit

- **Mandatory:** log denied write attempts and successful sensitive writes (post/reverse/delete).
- **Optional / pack-tier:** log group-level read/report access.

### FIN-HOLD-L16 — Org depth caps (Platform Owner)

- Max company-tree depth and branch-tree depth are **feature-pack / Platform Owner** concerns (**DEBT-ORG-012**).
- Not blocking design of L01–L15 under full-pack assumption; must be enforced before unlimited production depth.

---

## 2. Non-goals

- Branch as a separate legal tenant.
- Department-level document series.
- Unlimited mega-holding on-prem deployment model.
- Silent preservation of `is_owner` full write across all companies.

---

## 3. Compliance gap vs current code (snapshot 2026-10-14)

| Area | Current | Required by this ADR |
|------|---------|----------------------|
| `FinanceCompanyScope` + `is_owner` | Group-wide **read and write** | Write limited to operate scope; owner ≠ auto operate-all |
| Journal create for child as owner | Allowed | Denied unless explicit operate grant |
| CoA | Tenant-shared | Per company |
| Branch series | Not implemented | Optional from fiscal year start |
| Group observer role | Missing as product surface | Required |
| Cross-module same law | Partial | Mandatory |

---

## 4. Implementation work breakdown (ordered)

See companion section in chat / track as **FIN-HOLD-W*** tasks.

| Wave | Focus |
|------|--------|
| W0 | Identity: permissions + group observer role; grant UI by parent |
| W1 | BE: split `assertCanReadCompany` vs `assertCanOperateCompany`; fix all FIN services |
| W2 | FE: remove child-company create for pure observers; company filter for read |
| W3 | Numbering: company setting `branch_series_from_fiscal_year`; sequence key extension |
| W4 | CoA per company migration + API |
| W5 | Org/Identity role seed audit (L12) |
| W6 | Audit events (L15) |
| W7 | Roll same guards to non-FIN ops modules |
| W8 | DEBT-ORG-012 depth caps (Platform Owner) |

---

## 5. Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-10-14 | Owner decisions L01–L16 locked from product workshop |
