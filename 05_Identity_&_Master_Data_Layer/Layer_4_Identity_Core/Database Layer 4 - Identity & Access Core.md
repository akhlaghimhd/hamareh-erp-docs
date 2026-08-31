# Database Layer 4 - Identity & Access Core

- **Version:** 1.2
- **Last Updated:** 2026-08-31
- **Category:** Identity & Master Data Layer
- **Status:** Approved
- **Official layer name:** Identity & Access Core
- **Code module:** `App\Modules\IdentityCore`
- **Source:** HamarehERP Architecture Documentation
- **SSOT:** `ADD_Layer_Module_Code_Mapping_v1.0.md`

> Former title «ERP Core Identity Organization Layer» is retired.  
> **Organization (Company/Branch/Department) is Layer 5** — see `App\Modules\Organization` and ERP Foundation.

---

## Decision Log – P4-0.1 (2026-08-31)

**Official Decision – Source of Truth for RBAC & Scope tables**

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

This decision aligns the documentation with the running code, prevents dual-table risk, and keeps the architecture consistent with the project’s Multi-Tenancy rules.

---

-- =========================================================================

-- Layer 4: Identity & Access Core (REVISED & SECURED)

-- =========================================================================

-- 1. users

CREATE TABLE users (

 user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 mobile VARCHAR(50),

 first_name VARCHAR(100),

 last_name VARCHAR(100),

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

-- 4. tenant_users

CREATE TABLE tenant_users (

 tenant_user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE RESTRICT,

 username VARCHAR(100) NOT NULL,

 tenant_user_email VARCHAR(150) NOT NULL,

 user_type SMALLINT NOT NULL DEFAULT 1,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_tenant_users_username ON tenant_users(tenant_id, username) WHERE deleted_at IS NULL;

CREATE UNIQUE INDEX uq_tenant_users_email ON tenant_users(tenant_id, tenant_user_email) WHERE deleted_at IS NULL;

CREATE INDEX idx_tenant_users_composite ON tenant_users(tenant_id, user_id) WHERE deleted_at IS NULL;

-- ============================================================================

-- NOTE (P4-0.1):
-- The following unprefixed tables (roles, permissions, role_permissions,
-- user_roles, scopes, user_scope_assignments) are retained in this document
-- only for historical reference. They were NEVER created in the real database.
-- The only valid and implemented tables are the tenant_* tables below.
-- ============================================================================

-- 5. roles  (HISTORICAL / NOT IMPLEMENTED – see Decision Log P4-0.1)

CREATE TABLE roles (

 role_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 name VARCHAR(100) NOT NULL,

 code VARCHAR(100) NOT NULL,

 description VARCHAR(500),

 role_type SMALLINT NOT NULL DEFAULT 1,

 role_scope_type SMALLINT NOT NULL DEFAULT 1,

 is_system_role BOOLEAN NOT NULL DEFAULT FALSE,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_tenant_role_code ON roles(tenant_id, code) WHERE deleted_at IS NULL;

-- 6. permissions  (HISTORICAL / NOT IMPLEMENTED – see Decision Log P4-0.1)

CREATE TABLE permissions (

 permission_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 code VARCHAR(150) NOT NULL,

 name VARCHAR(200) NOT NULL,

 module_name VARCHAR(100) NOT NULL,

 group_name VARCHAR(100),

 description VARCHAR(500),

 permission_type SMALLINT NOT NULL DEFAULT 1,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_permissions_code ON permissions(code) WHERE deleted_at IS NULL;

-- 7. role_permissions  (HISTORICAL / NOT IMPLEMENTED – see Decision Log P4-0.1)

CREATE TABLE role_permissions (

 role_permission_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 role_id UUID NOT NULL REFERENCES roles(role_id) ON DELETE RESTRICT,

 permission_id UUID NOT NULL REFERENCES permissions(permission_id) ON DELETE RESTRICT,

 access_level SMALLINT NOT NULL DEFAULT 1,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_role_permission ON role_permissions(role_id, permission_id) WHERE deleted_at IS NULL;

-- 8. user_roles  (HISTORICAL / NOT IMPLEMENTED – see Decision Log P4-0.1)

CREATE TABLE user_roles (

 user_role_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_user_id UUID NOT NULL REFERENCES tenant_users(tenant_user_id) ON DELETE RESTRICT,

 role_id UUID NOT NULL REFERENCES roles(role_id) ON DELETE RESTRICT,

 assigned_by UUID,

 start_date TIMESTAMPTZ,

 end_date TIMESTAMPTZ,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_user_role ON user_roles(tenant_user_id, role_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_user_roles_role_lookup ON user_roles(role_id) WHERE deleted_at IS NULL;

-- 9. scopes  (HISTORICAL / NOT IMPLEMENTED – see Decision Log P4-0.1)

CREATE TABLE scopes (

 scope_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 scope_type SMALLINT NOT NULL,

 reference_type VARCHAR(100) NOT NULL,

 reference_id UUID NOT NULL,

 name VARCHAR(200) NOT NULL,

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

CREATE INDEX idx_scopes_tenant ON scopes(tenant_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_scopes_polymorphic ON scopes(reference_type, reference_id) WHERE deleted_at IS NULL;

-- 10. user_scope_assignments  (HISTORICAL / NOT IMPLEMENTED – see Decision Log P4-0.1)

CREATE TABLE user_scope_assignments (

 user_scope_assignment_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_user_id UUID NOT NULL REFERENCES tenant_users(tenant_user_id) ON DELETE RESTRICT,

 scope_id UUID NOT NULL REFERENCES scopes(scope_id) ON DELETE RESTRICT,

 is_primary BOOLEAN NOT NULL DEFAULT FALSE,

 start_date TIMESTAMPTZ,

 end_date TIMESTAMPTZ,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_user_scope ON user_scope_assignments(tenant_user_id, scope_id) WHERE deleted_at IS NULL;

-- ============================================================================

-- OFFICIAL & IMPLEMENTED TABLES (Source of Truth – Decision P4-0.1)
-- Tenant Level RBAC Security Model (tenant_* tables)
-- These are the only tables that exist in the real database and codebase.
-- ============================================================================

CREATE TABLE tenant_roles (

    tenant_role_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    code VARCHAR(100) NOT NULL,

    name VARCHAR(150) NOT NULL,

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

CREATE UNIQUE INDEX uq_tenant_roles_code
ON tenant_roles(tenant_id, code)
WHERE deleted_at IS NULL;

CREATE INDEX idx_tenant_roles_tenant
ON tenant_roles(tenant_id)
WHERE deleted_at IS NULL;

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

CREATE UNIQUE INDEX uq_tenant_permissions_code
ON tenant_permissions(tenant_id, code)
WHERE deleted_at IS NULL;

CREATE INDEX idx_tenant_permissions_module
ON tenant_permissions(tenant_id, module_name)
WHERE deleted_at IS NULL;

CREATE TABLE tenant_user_roles (

    tenant_user_role_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    user_id UUID NOT NULL,

    tenant_role_id UUID NOT NULL
        REFERENCES tenant_roles(tenant_role_id)
        ON DELETE RESTRICT,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_tenant_user_roles
ON tenant_user_roles(
    tenant_id,
    user_id,
    tenant_role_id
)
WHERE deleted_at IS NULL;

CREATE INDEX idx_tenant_user_roles_user
ON tenant_user_roles(
    tenant_id,
    user_id
)
WHERE deleted_at IS NULL;

CREATE TABLE tenant_role_permissions (

    tenant_role_permission_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    tenant_role_id UUID NOT NULL
        REFERENCES tenant_roles(tenant_role_id)
        ON DELETE RESTRICT,

    tenant_permission_id UUID NOT NULL
        REFERENCES tenant_permissions(tenant_permission_id)
        ON DELETE RESTRICT,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_tenant_role_permissions
ON tenant_role_permissions(
    tenant_id,
    tenant_role_id,
    tenant_permission_id
)
WHERE deleted_at IS NULL;

CREATE INDEX idx_tenant_role_permissions_role
ON tenant_role_permissions(
    tenant_role_id
)
WHERE deleted_at IS NULL;
