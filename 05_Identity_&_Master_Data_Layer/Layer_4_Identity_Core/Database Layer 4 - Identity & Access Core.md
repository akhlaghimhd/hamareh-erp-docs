# Database Layer 4 - Identity & Access Core

- **Version:** 1.4
- **Last Updated:** 2026-09-09
- **Category:** Identity & Master Data Layer
- **Status:** Approved
- **Official layer name:** Identity & Access Core
- **Code module:** `App\Modules\IdentityCore`
- **Source:** HamarehERP Architecture Documentation
- **SSOT:** `ADD_Layer_Module_Code_Mapping_v1.0.md`

> Former title «ERP Core Identity Organization Layer» is retired.  
> **Organization (Company/Branch/Department) is Layer 5** — see `App\Modules\Organization` and ERP Foundation.

---

## Decision Log – Phase 0 (Layer 4)

### P4-0.1 (2026-08-31) – Source of Truth for RBAC & Scope tables

In the initial version of this document, RBAC and Scope tables were designed with unprefixed names (`roles`, `permissions`, `role_permissions`, `user_roles`, `scopes`, `user_scope_assignments`).

During actual implementation, due to strict Multi-Tenancy requirements (TenantContext + RLS + Law 1.1 / 1.2), these tables were implemented with the `tenant_` prefix:

- `tenant_roles`
- `tenant_permissions`
- `tenant_user_roles`
- `tenant_role_permissions`
- `tenant_scopes`
- `tenant_user_scopes`

**Final and binding decision:**

1. The tables that currently exist and are used in the codebase and database (`tenant_*`) are the **only official and valid Source of Truth**.
2. The unprefixed table names that appear only in the early design of this document **were never created in the real database**. They belong solely to the initial design draft.
3. From this moment forward, all development, migrations, tests, seeders, and documentation must be based exclusively on the `tenant_*` tables.
4. Unprefixed names are no longer valid. No new tables must be created based on them.

### P4-0.2 (2026-08-31) – Official Scope naming mapping

| Concept in early SSOT | Official name in code & DB (SoT) |
|-----------------------|----------------------------------|
| scopes                | `tenant_scopes`                  |
| user_scope_assignments| `tenant_user_scopes`             |
| Model                 | `TenantScope` / `TenantUserScope`|
| Service               | `ScopeService`                   |

All future references, migrations, tests and documentation must use the `tenant_*` names only.

### P4-0.3 (2026-08-31) – Ownership boundary

- **IdentityCore (Layer 4)** owns: users, credentials, profiles, tenant_users, RBAC (`tenant_roles` / `tenant_permissions` / …), Scopes (`tenant_scopes` / `tenant_user_scopes`), membership history.
- **Organization (Layer 5)** owns: Company, Branch, Department and related structural entities.
- No ownership transfer. Cross-module references remain logical only (no physical FK between modules).

### P4-0.4 (2026-08-31) – Minimum required CRUD surface

| Entity              | Required operations                                      |
|---------------------|----------------------------------------------------------|
| User / TenantUser   | list, show, create, update, soft-delete (+ row_version) |
| Role                | list, show, create, update, soft-delete, assign to user, assign permissions |
| Permission          | list, show, create, update, soft-delete                 |
| Scope               | list, show, create, update, soft-delete, assign / unassign to user |
| UserProfile         | show, create/update (upsert), soft-delete               |
| MembershipHistory   | append + read (no hard delete)                          |

All operations must respect Soft Delete, `row_version`, tenant isolation and permission middleware.

### P4-0.5 (2026-09-09) – Schema alignment with implemented code

Code migrations under `database/migrations/identity_core/` are binding for column sets:

- `users` includes optional `email VARCHAR(255) UNIQUE` (login identifier).
- `tenant_users` uses `is_owner`, `employee_id` (no `username` / `tenant_user_email` / `user_type` from early draft).
- `tenant_scopes` uses `scope_name`, `scope_type` (string), `reference_id`, `is_active`.
- `tenant_user_scopes` PK is `assignment_id`.
- `tenant_membership_histories` is official and required.
- `sessions` under identity_core migrations is accepted Laravel infrastructure, not a domain entity.

---

-- =========================================================================
-- Layer 4: Identity & Access Core (REVISED & SECURED) – implemented SoT
-- =========================================================================

-- 1. users
CREATE TABLE users (
 user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
 mobile VARCHAR(50),
 first_name VARCHAR(100),
 last_name VARCHAR(100),
 email VARCHAR(255) UNIQUE,
 user_kind SMALLINT NOT NULL DEFAULT 1,
 status SMALLINT NOT NULL DEFAULT 1,
 last_login_at TIMESTAMPTZ,
 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
 created_by UUID,
 updated_at TIMESTAMPTZ,
 updated_by UUID,
 deleted_at TIMESTAMPTZ,
 deleted_by UUID,
 row_version BIGINT NOT NULL DEFAULT 1
);

-- 2. user_credentials
CREATE TABLE user_credentials (
 credential_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
 user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE RESTRICT,
 password_hash VARCHAR(500),
 authentication_type SMALLINT NOT NULL DEFAULT 1,
 is_verified BOOLEAN NOT NULL DEFAULT FALSE,
 two_factor_enabled BOOLEAN NOT NULL DEFAULT FALSE,
 failed_login_count INT NOT NULL DEFAULT 0,
 locked_until TIMESTAMPTZ,
 last_password_change_at TIMESTAMPTZ,
 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
 created_by UUID,
 updated_at TIMESTAMPTZ,
 updated_by UUID,
 deleted_at TIMESTAMPTZ,
 deleted_by UUID,
 row_version BIGINT NOT NULL DEFAULT 1
);
CREATE INDEX idx_user_credentials_user ON user_credentials(user_id) WHERE deleted_at IS NULL;

-- 3. user_profiles
CREATE TABLE user_profiles (
 profile_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
 user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE RESTRICT,
 national_id VARCHAR(50),
 birth_date DATE,
 avatar_url VARCHAR(500),
 gender SMALLINT,
 address TEXT,
 phone VARCHAR(50),
 description VARCHAR(500),
 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
 created_by UUID,
 updated_at TIMESTAMPTZ,
 updated_by UUID,
 deleted_at TIMESTAMPTZ,
 deleted_by UUID,
 row_version BIGINT NOT NULL DEFAULT 1
);
CREATE INDEX idx_user_profiles_user ON user_profiles(user_id) WHERE deleted_at IS NULL;

-- 4. tenant_users (implemented shape)
CREATE TABLE tenant_users (
 tenant_user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
 tenant_id UUID NOT NULL,
 user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE RESTRICT,
 employee_id UUID,
 is_owner BOOLEAN NOT NULL DEFAULT FALSE,
 status SMALLINT NOT NULL DEFAULT 1,
 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
 created_by UUID,
 updated_at TIMESTAMPTZ,
 updated_by UUID,
 deleted_at TIMESTAMPTZ,
 deleted_by UUID,
 row_version BIGINT NOT NULL DEFAULT 1
);
CREATE UNIQUE INDEX uq_tenant_users ON tenant_users(tenant_id, user_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_tenant_users_employee ON tenant_users(tenant_id, employee_id) WHERE deleted_at IS NULL;

-- ============================================================================
-- OFFICIAL & IMPLEMENTED TABLES (Source of Truth – Decision P4-0.1)
-- ============================================================================

CREATE TABLE tenant_roles (
    tenant_role_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    code VARCHAR(100) NOT NULL,
    name VARCHAR(150) NOT NULL,
    description VARCHAR(500),
    is_system_default BOOLEAN NOT NULL DEFAULT FALSE,
    status SMALLINT NOT NULL DEFAULT 1,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by UUID,
    updated_at TIMESTAMPTZ,
    updated_by UUID,
    deleted_at TIMESTAMPTZ,
    deleted_by UUID,
    row_version BIGINT NOT NULL DEFAULT 1
);
CREATE UNIQUE INDEX uq_tenant_roles_code ON tenant_roles(tenant_id, code) WHERE deleted_at IS NULL;
CREATE INDEX idx_tenant_roles_tenant ON tenant_roles(tenant_id) WHERE deleted_at IS NULL;

CREATE TABLE tenant_permissions (
    tenant_permission_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    code VARCHAR(150) NOT NULL,
    name VARCHAR(200) NOT NULL,
    module_name VARCHAR(100),
    action_type VARCHAR(50),
    description VARCHAR(500),
    status SMALLINT NOT NULL DEFAULT 1,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by UUID,
    updated_at TIMESTAMPTZ,
    updated_by UUID,
    deleted_at TIMESTAMPTZ,
    deleted_by UUID,
    row_version BIGINT NOT NULL DEFAULT 1
);
CREATE UNIQUE INDEX uq_tenant_permissions_code ON tenant_permissions(tenant_id, code) WHERE deleted_at IS NULL;
CREATE INDEX idx_tenant_permissions_module ON tenant_permissions(tenant_id, module_name) WHERE deleted_at IS NULL;

CREATE TABLE tenant_user_roles (
    tenant_user_role_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    user_id UUID NOT NULL,
    tenant_role_id UUID NOT NULL REFERENCES tenant_roles(tenant_role_id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by UUID,
    updated_at TIMESTAMPTZ,
    updated_by UUID,
    deleted_at TIMESTAMPTZ,
    deleted_by UUID,
    row_version BIGINT NOT NULL DEFAULT 1
);
CREATE UNIQUE INDEX uq_tenant_user_roles ON tenant_user_roles(tenant_id, user_id, tenant_role_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_tenant_user_roles_user ON tenant_user_roles(tenant_id, user_id) WHERE deleted_at IS NULL;

CREATE TABLE tenant_role_permissions (
    tenant_role_permission_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    tenant_role_id UUID NOT NULL REFERENCES tenant_roles(tenant_role_id) ON DELETE RESTRICT,
    tenant_permission_id UUID NOT NULL REFERENCES tenant_permissions(tenant_permission_id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by UUID,
    updated_at TIMESTAMPTZ,
    updated_by UUID,
    deleted_at TIMESTAMPTZ,
    deleted_by UUID,
    row_version BIGINT NOT NULL DEFAULT 1
);
CREATE UNIQUE INDEX uq_tenant_role_permissions ON tenant_role_permissions(tenant_id, tenant_role_id, tenant_permission_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_tenant_role_permissions_role ON tenant_role_permissions(tenant_role_id) WHERE deleted_at IS NULL;

CREATE TABLE tenant_scopes (
    scope_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    scope_name VARCHAR(150) NOT NULL,
    scope_type VARCHAR(50) NOT NULL,
    reference_id UUID,
    description TEXT,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by UUID,
    updated_at TIMESTAMPTZ,
    updated_by UUID,
    deleted_at TIMESTAMPTZ,
    deleted_by UUID,
    row_version BIGINT NOT NULL DEFAULT 1
);
CREATE UNIQUE INDEX uq_tenant_scopes_reference ON tenant_scopes(tenant_id, scope_type, reference_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_tenant_scopes_type ON tenant_scopes(tenant_id, scope_type) WHERE deleted_at IS NULL;

CREATE TABLE tenant_user_scopes (
    assignment_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    tenant_user_id UUID NOT NULL REFERENCES tenant_users(tenant_user_id) ON DELETE RESTRICT,
    scope_id UUID NOT NULL REFERENCES tenant_scopes(scope_id) ON DELETE RESTRICT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by UUID,
    updated_at TIMESTAMPTZ,
    updated_by UUID,
    deleted_at TIMESTAMPTZ,
    deleted_by UUID,
    row_version BIGINT NOT NULL DEFAULT 1
);
CREATE UNIQUE INDEX uq_tenant_user_scopes ON tenant_user_scopes(tenant_id, tenant_user_id, scope_id) WHERE deleted_at IS NULL;

CREATE TABLE tenant_membership_histories (
    history_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    tenant_user_id UUID NOT NULL REFERENCES tenant_users(tenant_user_id) ON DELETE RESTRICT,
    previous_status SMALLINT,
    new_status SMALLINT NOT NULL,
    reason_code VARCHAR(50),
    description TEXT,
    effective_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by UUID,
    updated_at TIMESTAMPTZ,
    updated_by UUID,
    deleted_at TIMESTAMPTZ,
    deleted_by UUID,
    row_version BIGINT NOT NULL DEFAULT 1
);
CREATE INDEX idx_tenant_membership_histories_user ON tenant_membership_histories(tenant_id, tenant_user_id) WHERE deleted_at IS NULL;

-- sessions: Laravel session store under identity_core migrations – accepted infrastructure (not domain entity).
