# ADD — Scope Enforcement Policy (F2)

- **Version:** 1.0
- **Date:** 2026-08-29
- **Status:** Approved (Layer 2 Identity & Security)
- **Related:** Tenant Isolation Architecture Standard (Freeze), ERP SaaS Core Identity Database Design v1.0, Law 4.2 / 4.3

---

## 1. Decision

Scope resource filtering is **configurable** via application config:

| Mode | Code value | Behaviour when user has no scopes of the model `scopeType` |
|------|------------|------------------------------------------------------------|
| **Policy B — Gradual (default)** | `gradual` | No extra filter; tenant isolation only |
| **Policy A — Strict (target)** | `strict` | Deny all rows (`WHERE 1 = 0`) for types in `strict_scope_types` |

**Config path:** `config/scope.php`  
**Env:** `SCOPE_ENFORCEMENT_MODE=gradual|strict`  
**Default:** `gradual`  
**Strict types (when mode is strict):** `COMPANY`, `BRANCH`, `WAREHOUSE`

## 2. Rationale

- Identity Design: Scope is the heart of data security; evaluation flow ends with **Validate Scope → Execute Action** (fail-closed target).
- Tenant Isolation Standard: mandates tenant_id + RLS; is silent on “no scopes of a given type”.
- Current codebase already implemented gradual behaviour; immediate strict would lock out users without Scope assignment.
- Configurable policy preserves the architectural end-state (A) while allowing safe rollout (B).

## 3. Shared rules (both policies)

1. If the user has one or more scopes of the model `scopeType` → filter to allowed `reference_id`s.
2. If scopes of that type exist but `reference_id` list is empty → **zero rows**.
3. Tenant isolation (`TenantScoped` + RLS) always remains in force.
4. `withoutScopeIsolation()` may bypass only the Scope global scope (not tenant isolation).

## 4. Migration path to Policy A

1. Complete Scope assignment for operational users (COMPANY / BRANCH / WAREHOUSE).
2. Ensure Owner/Admin receive full or explicit scopes as required by product rules.
3. Deliver **F3** (Validate Scope before Action on scoped resources).
4. Switch env: `SCOPE_ENFORCEMENT_MODE=strict` and verify isolation tests in CI.

## 5. Implementation references

- Trait: `app/Base/Traits/ScopeScoped.php`
- Config: `config/scope.php`
- Tests: `tests/Feature/Modules/IdentityCore/ScopeIsolationTest.php`
- Middleware loading scopes: `app/Base/Http/Middleware/LoadUserScopesMiddleware.php`

## 6. Non-goals of this ADD

- Does not replace F3 (pre-action Scope validation).
- Does not change JWT vs DB source of truth (F4).
- Does not implement Isolation §10 full suite (F6).
