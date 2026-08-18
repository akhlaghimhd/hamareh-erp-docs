# ERP SaaS Database Design Roadmap Phase 1

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** System Blueprint & Roadmaps
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- ============================================================================

-- Phase 1 - Core ERP Foundation (REVISED) - PART 1 OF 4

-- Database Engine: PostgreSQL 18

-- ============================================================================

CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- 1. erp_companies

CREATE TABLE erp_companies (

 company_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 legal_name VARCHAR(200) NOT NULL,

 display_name VARCHAR(200),

 company_type SMALLINT NOT NULL DEFAULT 1,

 national_id VARCHAR(50),

 economic_code VARCHAR(50),

 registration_number VARCHAR(100),

 is_legal_entity BOOLEAN NOT NULL DEFAULT FALSE,

 phone VARCHAR(50),

 email VARCHAR(150),

 address TEXT,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_erp_companies_national_id ON erp_companies(tenant_id, national_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_erp_companies_tenant_id ON erp_companies(tenant_id) WHERE deleted_at IS NULL;

-- 2. erp_branches

CREATE TABLE erp_branches (

 branch_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 company_id UUID NOT NULL REFERENCES erp_companies(company_id) ON DELETE RESTRICT,

 parent_branch_id UUID REFERENCES erp_branches(branch_id) ON DELETE RESTRICT,

 code VARCHAR(50) NOT NULL,

 name VARCHAR(200) NOT NULL,

 is_main_branch BOOLEAN NOT NULL DEFAULT FALSE,

 phone VARCHAR(50),

 address TEXT,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_branch_company_code ON erp_branches(company_id, code) WHERE deleted_at IS NULL;

-- 3. erp_departments

CREATE TABLE erp_departments (

 department_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 company_id UUID NOT NULL REFERENCES erp_companies(company_id) ON DELETE RESTRICT,

 branch_id UUID REFERENCES erp_branches(branch_id) ON DELETE RESTRICT,

 parent_department_id UUID REFERENCES erp_departments(department_id) ON DELETE RESTRICT,

 code VARCHAR(50) NOT NULL,

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

CREATE UNIQUE INDEX uq_department_company_code ON erp_departments(company_id, code) WHERE deleted_at IS NULL;

-- 4. users (کاربران فراتننتی عمومی پلتفرم)

CREATE TABLE users (

 user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 username VARCHAR(100),

 email VARCHAR(150),

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

CREATE INDEX idx_users_status ON users(status) WHERE deleted_at IS NULL;

-- ============================================================================

-- Phase 1 - Core ERP Foundation (REVISED) - PART 2 OF 4

-- Database Engine: PostgreSQL 18

-- ============================================================================

-- 5. user_credentials

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

-- 6. user_profiles

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

-- 7. tenant_users (پل ارتباطی مالتی-تننت با لایه هویت)

-- نکته حیاتی: فیلد ایمیل و نام کاربری به این لایه منتقل و با tenant_id ترکیب یکتا شدند.

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

CREATE INDEX idx_tenant_users_tenant ON tenant_users(tenant_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_tenant_users_user ON tenant_users(user_id) WHERE deleted_at IS NULL;

-- 8. roles

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

CREATE UNIQUE INDEX uq_roles_tenant_code ON roles(tenant_id, code) WHERE deleted_at IS NULL;

CREATE INDEX idx_roles_tenant ON roles(tenant_id) WHERE deleted_at IS NULL;

-- ============================================================================

-- Phase 1 - Core ERP Foundation (REVISED) - PART 3 OF 4

-- Database Engine: PostgreSQL 18

-- ============================================================================

-- 9. permissions

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

CREATE INDEX idx_permissions_module ON permissions(module_name) WHERE deleted_at IS NULL;

-- 10. role_permissions

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

CREATE UNIQUE INDEX uq_role_permissions_composite ON role_permissions(role_id, permission_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_role_permissions_role ON role_permissions(role_id) WHERE deleted_at IS NULL;

-- 11. user_roles

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

CREATE UNIQUE INDEX uq_user_roles_composite ON user_roles(tenant_user_id, role_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_user_roles_tenant_user ON user_roles(tenant_user_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_user_roles_role ON user_roles(role_id) WHERE deleted_at IS NULL;

-- 12. scopes

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

CREATE INDEX idx_scopes_reference ON scopes(reference_type, reference_id) WHERE deleted_at IS NULL;

-- 13. user_scope_assignments

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

CREATE UNIQUE INDEX uq_user_scope_assignment ON user_scope_assignments(tenant_user_id, scope_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_user_scope_assignment_user ON user_scope_assignments(tenant_user_id) WHERE deleted_at IS NULL;

-- ============================================================================

-- Phase 1 - Core ERP Foundation (REVISED) - PART 4 OF 4

-- Database Engine: PostgreSQL 18

-- ============================================================================

-- 14. organizational_unit_types

CREATE TABLE organizational_unit_types (

 organizational_unit_type_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 code VARCHAR(50) NOT NULL,

 name VARCHAR(100) NOT NULL,

 description VARCHAR(500),

 level_order SMALLINT NOT NULL DEFAULT 1,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_organizational_unit_types_code ON organizational_unit_types(code) WHERE deleted_at IS NULL;

CREATE INDEX idx_organizational_unit_types_status ON organizational_unit_types(status) WHERE deleted_at IS NULL;

-- 15. organizational_units

CREATE TABLE organizational_units (

 organizational_unit_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 company_id UUID NOT NULL REFERENCES erp_companies(company_id) ON DELETE RESTRICT,

 parent_organizational_unit_id UUID REFERENCES organizational_units(organizational_unit_id) ON DELETE RESTRICT,

 organizational_unit_type_id UUID NOT NULL REFERENCES organizational_unit_types(organizational_unit_type_id) ON DELETE RESTRICT,

 code VARCHAR(50) NOT NULL,

 name VARCHAR(200) NOT NULL,

 manager_user_id UUID REFERENCES users(user_id) ON DELETE SET NULL, -- ارجاع منطقی مجاز بر اساس اصول مرزبندی

 description VARCHAR(500),

 level_order SMALLINT NOT NULL DEFAULT 1,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_organizational_units_company_code ON organizational_units(company_id, code) WHERE deleted_at IS NULL;

CREATE INDEX idx_organizational_units_company ON organizational_units(company_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_organizational_units_parent ON organizational_units(parent_organizational_unit_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_organizational_units_type ON organizational_units(organizational_unit_type_id) WHERE deleted_at IS NULL;

-- 16. master_data_categories

CREATE TABLE master_data_categories (

 master_data_category_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 code VARCHAR(100) NOT NULL,

 name VARCHAR(200) NOT NULL,

 description VARCHAR(500),

 is_system_category BOOLEAN NOT NULL DEFAULT FALSE,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_master_data_categories_tenant_code ON master_data_categories(tenant_id, code) WHERE deleted_at IS NULL;

CREATE INDEX idx_master_data_categories_tenant ON master_data_categories(tenant_id) WHERE deleted_at IS NULL;

-- 17. master_data_values

CREATE TABLE master_data_values (

 master_data_value_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 master_data_category_id UUID NOT NULL REFERENCES master_data_categories(master_data_category_id) ON DELETE RESTRICT,

 parent_master_data_value_id UUID REFERENCES master_data_values(master_data_value_id) ON DELETE RESTRICT,

 code VARCHAR(100) NOT NULL,

 name VARCHAR(200) NOT NULL,

 sort_order INT NOT NULL DEFAULT 0,

 extra_data JSONB, -- پشتیبانی از ویژگی‌های داینامیک انبار، کالا و شرکای تجاری

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_master_data_values_category_code ON master_data_values(master_data_category_id, code) WHERE deleted_at IS NULL;

CREATE INDEX idx_master_data_values_category ON master_data_values(master_data_category_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_master_data_values_extra_data_gin ON master_data_values USING GIN (extra_data); -- ایندکس GIN برای کارایی فوق‌العاده بالا روی ویژگی‌های داینامیک JSONB

-- 18. currencies

-- اصلاح استراتژیک بر اساس قوانین حاکمیت داده: کاتالوگ ارزها کاملاً عمومی و گلوبال (پلتفرمی) شد.

CREATE TABLE currencies (

 currency_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 code VARCHAR(10) NOT NULL,

 name VARCHAR(100) NOT NULL,

 symbol VARCHAR(20),

 decimal_places SMALLINT NOT NULL DEFAULT 2,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_currencies_code ON currencies(code) WHERE deleted_at IS NULL;

-- 19. tenant_currencies (جدول پیوند جدید مالتی-تننت برای پایش ارز پایه هر شرکت بدون نشت داده)

CREATE TABLE tenant_currencies (

 tenant_id UUID NOT NULL, -- ارجاع منطقی لایه‌ای به لایه ساس

 currency_id UUID NOT NULL REFERENCES currencies(currency_id) ON DELETE RESTRICT,

 is_base_currency BOOLEAN NOT NULL DEFAULT FALSE,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 PRIMARY KEY (tenant_id, currency_id)

);

CREATE INDEX idx_tenant_currencies_lookup ON tenant_currencies(tenant_id) WHERE is_base_currency = TRUE;

