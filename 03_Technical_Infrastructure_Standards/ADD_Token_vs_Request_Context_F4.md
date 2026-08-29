# ADD — Token vs Request Context (F4)

- **Version:** 1.0
- **Date:** 2026-08-29
- **Status:** Approved (Layer 2 Identity & Security)
- **Related:** Law 4.4, ERP SaaS Core Identity Database Design v1.0, LoadUserScopesMiddleware, AuthenticationService

---

## 1. Decision

| Concern | Source of truth | Carrier |
|---------|-----------------|--------|
| **Authentication** (who is the user?) | Sanctum personal access token | `Authorization: Bearer` |
| **Tenant binding** (which tenant?) | Request header + DB membership check | `X-Tenant-ID` + `tenant_users` |
| **Authorization** (roles, permissions, scopes) | **Database at request time** | `LoadUserScopesMiddleware` → `ScopeContext` / `Context` / container |

**Forbidden:** Treating roles, permissions, or scopes embedded in the login response, JWT claims, or client-supplied headers/body as authoritative for access control.

## 2. Token responsibilities (Sanctum)

On login, token is issued with abilities such as:

- `*`
- `tenant:{tenant_id}` (when tenant is known)

The token proves **identity** (and optionally a tenant ability hint). It does **not** store the live role/permission/scope set for enforcement.

Login response may include a **snapshot** `security_context` for the client UI only. That snapshot is informational and may become stale.

## 3. Request Context lifecycle

For protected routes (`auth:sanctum` + `load.scopes`):

1. `TenantContextMiddleware` — validates `X-Tenant-ID`, membership, sets RLS `app.current_tenant_id`.
2. `auth:sanctum` — resolves authenticated user from token.
3. `LoadUserScopesMiddleware` — **reloads from DB**:
   - scopes (`tenant_user_scopes` + `tenant_scopes`)
   - roles (`tenant_user_roles` + `tenant_roles`)
   - permissions (`tenant_role_permissions` + `tenant_permissions`)
4. Fills `ScopeContext`, Laravel `Context`, and container bindings used by ScopeScoped / ScopeAccessGuard / services.
5. `RequirePermission` evaluates permission codes from **DB** (optionally cached under tenant tag; cache is derived data, not an independent authority).

## 4. Rules

1. Any change to role, permission, or scope assignment takes effect on the **next request** after DB commit (and after permission cache miss/invalidation if cache is used).
2. Clients must not send roles/scopes for authorization; server ignores such claims if present.
3. Background jobs must rebuild Tenant + Scope context from DB (or trusted job payload that is itself loaded from DB at enqueue time), not from a stale client token body.

## 5. Implementation references

- `app/Modules/IdentityCore/Services/AuthenticationService.php` — login loads context from DB; token abilities only identity/tenant.
- `app/Base/Http/Middleware/LoadUserScopesMiddleware.php` — per-request DB reload.
- `app/Base/Http/Middleware/RequirePermission.php` — permission check from DB (+ tenant-tagged cache).
- `app/Base/Services/ScopeAccessGuard.php` / `ScopeScoped` — use request ScopeContext, not token claims.

## 6. Test expectation

- Login returns `security_context` populated from DB (existing SecurityContextLoginTest).
- After revoking a permission in DB (and clearing permission cache), a subsequent API call with the **same** token is denied (F4 regression test).
