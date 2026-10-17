# ADR-ID-ORG-005 — Membership Anchor & Hierarchical Scope Ceiling

| Field | Value |
|-------|--------|
| **Status** | **ACCEPTED** |
| **Date** | 2026-10-17 |
| **Module** | IdentityCore + Organization (logical company tree) |
| **Supplements** | ADR-ID-ORG-003, ADR-ID-ORG-004, ADR-FIN-003 |

## 1. Context

Holding uses a **single tenant** and **Scope-based** authorization (ADR-ID-ORG-003).
Operational modules already split **group read ≠ operate** (ADR-FIN-003).

Residual risk: a user who obtains Identity permissions (`identity.scope.assign`, `identity.role.assign`, …)
could, without a **hard organizational ceiling**, pick peer or parent companies/scopes and expand
access toward the whole group — including to self or colleagues — beyond the company where
membership was defined.

Existing `assertCanGrantScopeIds` limits grants to the actor’s **current** COMPANY/BRANCH scopes.
That is necessary but not sufficient: list APIs may still expose the full tenant catalog, and there
is no durable **membership anchor** independent of later scope changes.

## 2. Decision (product laws)

### ID-CEILING-L01 — Membership anchor

Every `tenant_users` row **SHOULD** carry `home_company_id` (logical UUID → `erp_companies.company_id`):

- Set at membership **create** (from creator’s company context or explicit input).
- **Immutable** for the member via self-service or peer APIs.
- Change only by **technical_owner** / soft-SME path, or by an actor whose ceiling **strictly contains**
  the new home (ancestor of both old and new), with audit.

### ID-CEILING-L02 — Visibility ceiling (no peer / no upward)

A non–group-wide actor **MUST NOT** see, list, or select companies (or structural scopes bound to them)
that are **ancestors or siblings** of their effective ceiling roots.

**Effective ceiling roots** for an actor:

- Technical owner / soft SME / identity mutate group-wide → **no ceiling** (full tenant, existing rules).
- Otherwise → actor’s COMPANY scope reference_ids (and companies implied by BRANCH scopes),
  each expanded to **self + descendants** on `erp_companies.parent_company_id` (legal company tree).

Peers of those roots and any parent toward the holding root are **out of ceiling**.

### ID-CEILING-L03 — Grant only within ceiling

`assignScope` / equivalent APIs **MUST** reject any COMPANY/BRANCH scope whose resolved company
is outside the actor’s ceiling (L02).

### ID-CEILING-L04 — Grant to a member requires high-water authority

When granting scopes or elevating access for target membership `T`:

- Actor must be allowed to **manage** `T` (existing delegated-admin rules), **and**
- `T.home_company_id` (if set) must lie **inside** the actor’s ceiling (L02).

Only actors **at or above** the target’s home subtree may expand that member’s reach.
A peer company admin **MUST NOT** grant scopes for the target’s home or upward.

### ID-CEILING-L05 — Enforce in Service, not only UI

All of L02–L04 apply to **HTTP API and in-process service calls**. UI hiding alone is non-compliant.

### ID-CEILING-L06 — Exceptions (explicit)

| Actor | Ceiling |
|-------|---------|
| `technical_owner` (`is_owner`) | None for Identity mutate (unchanged IDORG-T03/T07) |
| Soft SME (≤1 company) | That single company |
| Group observer | Read group-wide; **no** scope/role grant |
| Company / branch operator | L02–L04 |

### ID-CEILING-L07 — Does not replace Scope

Authorization remains **Scope + Permission**. The company tree only **bounds which scopes may be
seen or granted**. Reports may still walk the legal tree for aggregation where FIN-HOLD allows read.

## 3. Consequences

### Positive

- Limits blast radius of over-granted Identity permissions to the actor’s company subtree.
- Aligns Identity admin with FIN-HOLD “no upward operate by default”.
- Auditable anchor independent of later scope edits.

### Trade-offs

- Multi-company legitimate users need **explicit** scopes under a ceiling granted by an ancestor.
- Matrix / shared-service staff require careful home + multi-scope grants from HQ.
- Backfill of `home_company_id` for existing rows required.

## 4. Implementation outline

| Step | Work |
|------|------|
| **C0** | Migration `home_company_id` on `tenant_users` + model |
| **C1** | `HoldingAccessService`: subtree helper, ceiling ids, tighten `assertCanGrantScopeIds` |
| **C2** | Manage-member asserts use target `home_company_id` |
| **C3** | `UserService::createTenantUser` sets home from actor company |
| **C4** | `ScopeService::listScopes` filter by ceiling |
| **C5** | Tests + optional FE company/scope pickers |
| **C6** | Backfill job / seeder for demo users |

## 5. Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-10-17 | Laws L01–L07 accepted; implementation started on develop |
