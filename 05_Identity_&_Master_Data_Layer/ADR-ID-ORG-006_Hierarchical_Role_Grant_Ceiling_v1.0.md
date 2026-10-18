# ADR-ID-ORG-006 — Hierarchical Role Grant Ceiling

| Field | Value |
|-------|--------|
| **Status** | **ACCEPTED** |
| **Date** | 2026-10-18 |
| **Module** | IdentityCore (Role assign) + Scope ceiling |
| **Supplements** | ADR-ID-ORG-003, ADR-ID-ORG-004, ADR-ID-ORG-005 |

## 1. Context

Scope ceiling (ADR-ID-ORG-005) bounds **which COMPANY/BRANCH scopes** an actor may see or grant.
It did **not** bound **which roles** may be assigned to a membership.

Resulting gap: a company/branch operator (or a parent admin acting on a child membership) could assign
`platform-admin` / group-tier roles to a user whose only scope is a child company or branch — privilege expansion.

Product decision (2026-10-18): **hard block (variant الف)** with an explicit elevation path.

## 2. Decision (product laws)

### ID-ROLE-L01 — Role grant tier

Every role code maps to a **grant tier**:

| Tier | Rank | Example codes |
|------|------|----------------|
| `branch` | 1 | `branch-admin`, `branch-*` |
| `company` | 2 | `company-operator`, `company-user`, `accountant`, most operational roles |
| `group` | 3 | `group-observer`, `org-multi-company`, `org-admin`, `identity-manager` |
| `tenant` | 4 | `platform-admin` |

Catalog groups (`is_group`) are not assignable as membership roles.

### ID-ROLE-L02 — Target capacity from scopes

For target user `U` in tenant:

| Condition | Target rank |
|-----------|-------------|
| `tenant_users.is_owner = true` | 4 (tenant) |
| Has COMPANY scope on **primary** or **root** (no parent) company | ≥ 3 (group) |
| Has any other COMPANY scope and/or `home_company_id` | ≥ 2 (company) |
| Only BRANCH scopes | 1 (branch) |
| Soft SME / single-company path with no explicit scopes | 2 (company) |

### ID-ROLE-L03 — Actor grant ceiling

| Actor | Max grant rank |
|-------|----------------|
| Technical owner / soft SME (`isIdentityMutateGroupWide`) | 4 |
| Group observer | **0** (cannot assign roles) |
| Company operator | 2 |
| Branch-scoped operator | 1 |

### ID-ROLE-L04 — Hard inequality (mandatory)

For every role **newly granted** to `U`:

```text
rank(role) <= min(actor_grant_rank, target_capacity_rank)
```

Violations **MUST** be rejected by the service layer (HTTP and in-process). UI filtering alone is non-compliant.

### ID-ROLE-L05 — Elevation path (only way “up”)

If a parent admin insists on a **group/tenant** role for a child/grandchild user:

1. **First** assign a COMPANY scope at **parent or ancestor** (or primary/root) to that user (existing scope-grant + ADR-ID-ORG-005 ceiling rules).
2. **Then** assign the higher-tier role (now target capacity rank is sufficient).

There is **no** silent bypass and **no** “warning only” path for ordinary assign.

### ID-ROLE-L06 — Persian errors

- Target too low: `برای اعطای این نقش، ابتدا باید اسکوپ سطح بالاتر (شرکت مادر یا گروه) به کاربر داده شود.`
- Actor too low: `این نقش خارج از سقف دسترسی شما برای اعطا است.`
- Observer: existing manage/deny paths.

### ID-ROLE-L07 — Does not replace Scope or Permission

Authorization remains Permission + Scope. This ADR only **bounds role assignment**.

## 3. Implementation notes

- SSOT mapping: `App\Modules\IdentityCore\Catalog\RoleGrantTier`
- Enforce in `RoleService::assignRoleToUser` (on **toAdd** only; existing roles may remain until explicitly revoked)
- Service: `RoleGrantCeilingService`
- Tests: `RoleGrantHierarchyTest`
- FE: hide/disable out-of-ceiling roles when metadata available; BE remains authority

## 4. Consequences

- Closes the `platform-admin` on child-scoped operator gap.
- Aligns role grant with org hierarchy without splitting the tenant role catalog.
- Requires operators to **raise scope before raising role** — intentional friction.
