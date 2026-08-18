# Database Layer 4 - ERP Core Identity Organization Layer

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Identity & Master Data Layer
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- =========================================================================

-- Layer 4: ERP Core Identity Layer (REVISED & SECURED)

-- =========================================================================

-- 1. users

-- بر اساس اصل ایزوله‌سازی، ایندکس یکتای گلوبال از روی ایمیل و نام‌کاربری حذف شد.

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

 user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE RESTRICT, -- تغییر به RESTRICT جهت امنیت داده

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

 user_id UUID NOT NULL REFERENCES users(user_id) ON DELETE RESTRICT, -- تغییر به RESTRICT

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

-- انتقال فیلدهای هویتی و ترکیب یکتا با tenant_id جهت جلوگیری از نشت داده

CREATE TABLE tenant_users (

 tenant_user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL, -- ارجاع منطقی به لایه ساس ادمین (بدون FK فیزیکی بر اساس اصول مرزبندی)

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

-- تضمین ۱۰۰٪ امنیت مالتی-تننتی: یک نام کاربری یا ایمیل فقط در سطح همان شرکت یکتاست.

CREATE UNIQUE INDEX uq_tenant_users_username ON tenant_users(tenant_id, username) WHERE deleted_at IS NULL;

CREATE UNIQUE INDEX uq_tenant_users_email ON tenant_users(tenant_id, tenant_user_email) WHERE deleted_at IS NULL;

CREATE INDEX idx_tenant_users_composite ON tenant_users(tenant_id, user_id) WHERE deleted_at IS NULL;

-- 5. roles

CREATE TABLE roles (

 role_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL, -- تغییر به NOT NULL برای اجباری بودن بافتار تننت

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

-- 6. permissions

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

-- 7. role_permissions

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

-- 8. user_roles

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

-- بهبود کارایی: شاخص سرچ سریع نقش‌ها برای موتور احراز دسترسی

CREATE INDEX idx_user_roles_role_lookup ON user_roles(role_id) WHERE deleted_at IS NULL;

-- 9. scopes

CREATE TABLE scopes (

 scope_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL, -- تغییر به NOT NULL بر اساس اصول امنیت چندمستأجری

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

-- بهبود کارایی: ایندکس ترکیبی چندشکلی (Polymorphic) برای لایه کنترل محدوده داده در بک‌آند

CREATE INDEX idx_scopes_polymorphic ON scopes(reference_type, reference_id) WHERE deleted_at IS NULL;

-- 10. user_scope_assignments

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

-- Layer 4: ERP Core Identity Organization Layer

-- Tenant Level RBAC Security Model

-- ============================================================================

-- 1. tenant_roles

-- نقش‌های کاربران داخل هر Tenant

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

-- ============================================================================

-- 2. tenant_permissions

-- مجوزهای دسترسی سطح Tenant

-- ============================================================================

CREATE TABLE tenant_permissions (

    tenant_permission_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    code VARCHAR(150) NOT NULL,

    name VARCHAR(200) NOT NULL,

    module_name VARCHAR(100),

    action_type VARCHAR(50),

    -- CREATE

    -- READ

    -- UPDATE

    -- DELETE

    -- APPROVE

    -- EXECUTE

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

-- ============================================================================

-- 3. tenant_user_roles

-- اتصال کاربران Tenant به نقش‌ها

-- ============================================================================

CREATE TABLE tenant_user_roles (

    tenant_user_role_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    user_id UUID NOT NULL,

    -- Logical Reference to Layer 4 Users

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

-- ============================================================================

-- 4. tenant_role_permissions

-- اتصال نقش‌ها به مجوزها

-- ============================================================================

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

