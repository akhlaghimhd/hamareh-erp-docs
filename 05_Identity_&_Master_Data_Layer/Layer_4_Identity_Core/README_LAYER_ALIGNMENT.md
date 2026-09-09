# Layer 4 alignment note

- **Official layer name:** Identity & Access Core
- **Layer number:** 4
- **Code module:** `App\Modules\IdentityCore`
- **Database file:** `Database Layer 4 - Identity & Access Core.md`
- **Scope:** Users, credentials, profiles, tenant membership, roles, permissions, scopes
- **Status (2026-09-09):** Closed to 100% for operational compliance; residual items below are **accepted deviations** (not open gaps).

**Organization (Company / Branch / Department) is Layer 5 – ERP Foundation**, code module `App\Modules\Organization`.

SSOT: `02_System_Blueprint_&_Roadmaps/ADD_Layer_Module_Code_Mapping_v1.0.md`

Retired filename: `Database Layer 4 - ERP Core Identity Organization Layer.md`

---

## Completion actions (2026-09-09)

| Code | Action |
|------|--------|
| L4-M01 / L4-M02 / L4-M03 / L4-M04 | Code schema is binding SoT (Decision P4-0.1). Document SQL section to be kept aligned with implemented migrations (users includes `email`; `tenant_users` uses `is_owner` + `employee_id`; `tenant_scopes` uses `scope_name` / `scope_type` string + `is_active`). |
| L4-M05 | Versioned Domain Event classes added under `App\Modules\IdentityCore\Events\`: `UserLoggedInV1`, `UserLoggedOutV1`, `UserCreatedV1`, `RoleAssignedV1`, `ScopeAssignedV1`. Outbox already used `identity.*.v1` types. |
| L4-M06 | Folder structure Domain/Application/Infrastructure/API **deferred project-wide** (same decision as Layer 2). Current flat structure (Controllers / DTOs / Models / Requests / Routes / Services / Events) is **accepted**. |
| L4-m01 | PK names `scope_id` / `assignment_id` are **accepted** (renaming would break FKs and data). |
| L4-m02 | `sessions` table under `identity_core` migrations is **accepted** (Laravel session store requirement). |
| L4-m03 | Cross-module communication via logical UUID references + event_outbox; no physical FK across modules. Explicit Interface contracts remain optional at Modular Monolith stage. |
| L4-m04 | Module registration is central: `App\Base\Providers\ModuleServiceProvider` auto-loads all module Routes. No per-module ServiceProvider required. |
| L4-m05 | API versioning confirmed: routes registered as `/api/v1/identity-core/...` by ModuleServiceProvider. |
| L4-V02 | `PermissionSeeder` updated with full Identity permission codes matching all IdentityCore route middleware. |
| L4-V01 | Tests use `#[Test]` Attribute; isolation/RLS/CRUD/outbox coverage present. Green status requires local Docker run confirmation by operator. |

## Official implemented tables (code SoT)

- `users` (includes `email`)
- `user_credentials`
- `user_profiles`
- `tenant_users` (`is_owner`, `employee_id`; no username/email columns)
- `tenant_roles`
- `tenant_permissions`
- `tenant_user_roles`
- `tenant_role_permissions`
- `tenant_scopes` (`scope_name`, `scope_type`, `reference_id`, `is_active`)
- `tenant_user_scopes` (PK `assignment_id`)
- `tenant_membership_histories`
- `sessions` (Laravel; accepted)

## Non-negotiables verified

- SoftDeletes + `deleted_by` + `row_version` on operational models
- `tenant_id` + RLS policies on all tenant-scoped tables
- No physical FK to other modules’ tables
- Core Only Once (Identity owned exclusively by IdentityCore)
