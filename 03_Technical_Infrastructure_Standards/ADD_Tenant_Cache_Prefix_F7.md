# ADD — Tenant Cache Prefix (F7)

- **Version:** 1.0
- **Date:** 2026-08-29
- **Status:** Approved (Layer 2 Identity & Security)
- **Related:** Tenant Isolation Architecture Standard §7

---

## Decision

All tenant-scoped cache entries MUST use:

```
tenant:{tenant_id}:{module}:{key}
```

Examples:

- `tenant:{uuid}:identity:user_permissions:{user_id}`
- `tenant:{uuid}:organization:branches`

## Implementation

- Helper: `App\Base\Support\TenantCache`
  - `key($module, $key, $tenantId = null)`
  - `remember(...)` / `forget(...)` with optional tenant tags for invalidation
- `RequirePermission` uses `TenantCache::remember('identity', "user_permissions:{userId}", ...)`

## Rules

1. Never store tenant data under a key that omits `tenant:{tenant_id}:`.
2. Prefer `TenantCache` over raw `Cache::` for any tenant-owned value.
3. Platform-wide (non-tenant) keys remain outside this pattern (e.g. system feature flags).
