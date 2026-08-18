# 01 hr management tables

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Vertical ERP Modules
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- ============================================================================

-- MODULE: HUMAN RESOURCES MANAGEMENT LAYER

-- Database Engine: PostgreSQL 18 Enterprise Configuration

-- File Identity: 01_hr_management_tables.sql

-- Path: 06_Vertical_ERP_Modules/06_HR_Module/

-- ============================================================================

CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- ============================================================================

-- employees

-- HR Master Data - Employee Identity and Employment Information

-- ============================================================================

CREATE TABLE employees (

    employee_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    business_partner_id UUID,

    -- ارتباط منطقی با Business Partner

    -- Employee می‌تواند Person در Business Partner باشد

    user_id UUID,

    -- ارتباط منطقی با Identity Layer

    employee_code VARCHAR(50) NOT NULL,

    employment_type SMALLINT NOT NULL DEFAULT 1,

    -- 1: Full Time

    -- 2: Part Time

    -- 3: Contract

    hire_date DATE NOT NULL,

    termination_date DATE,

    job_title VARCHAR(150),

    department_id UUID,

    -- Logical Reference to departments

    branch_id UUID,

    -- Logical Reference to branches

    status SMALLINT NOT NULL DEFAULT 1,

    -- 1: Active

    -- 2: Suspended

    -- 3: Terminated

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

-- Replace uq_employees_tenant_code index in 01_hr_management_tables.sql

CREATE UNIQUE INDEX uq_employees_tenant_code

    ON employees(tenant_id, employee_code)

    WHERE deleted_at IS NULL; -- Typo Fixed (deleted at -> deleted_at)

CREATE INDEX idx_employees_business_partner

ON employees(business_partner_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_employees_department

ON employees(department_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_employees_branch

ON employees(branch_id)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 1. employee_profiles (اطلاعات تکمیلی و پرونده پرسنلی متصل به موجودیت کارمند)

-- ============================================================================

CREATE TABLE employee_profiles (

    profile_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    employee_id UUID NOT NULL REFERENCES employees(employee_id) ON DELETE RESTRICT,

    marital_status SMALLINT NOT NULL DEFAULT 1, -- 1: Single, 2: Married

    emergency_contact_name VARCHAR(200),

    emergency_contact_phone VARCHAR(50),

    bank_name VARCHAR(150),

    bank_account_number VARCHAR(100),

    shaba_number VARCHAR(50), -- فرمت شبا IR

    education_level VARCHAR(100),

    home_address TEXT,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_hr_employee_profiles ON employee_profiles(tenant_id, employee_id) WHERE deleted_at IS NULL;

-- ============================================================================

-- 2. attendance_records (ثبت لایو و آنی تردد، ورود و خروج کارکنان)

-- ============================================================================

CREATE TABLE attendance_records (

    attendance_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    employee_id UUID NOT NULL REFERENCES employees(employee_id) ON DELETE RESTRICT,

    work_date DATE NOT NULL,

    clock_in TIMESTAMPTZ NOT NULL,

    clock_out TIMESTAMPTZ,

    device_id_in VARCHAR(100), -- متادیتای سخت‌افزار ثبت ساعت ورود

    device_id_out VARCHAR(100), -- متادیتای سخت‌افزار ثبت ساعت خروج

    attendance_status SMALLINT NOT NULL DEFAULT 1, -- 1: Present, 2: Late, 3: Absent, 4: Leave

    overtime_hours_approved NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ

);

CREATE INDEX idx_hr_attendance_lookup ON attendance_records(tenant_id, employee_id, work_date DESC);

CREATE INDEX idx_hr_attendance_unclosed ON attendance_records(tenant_id, employee_id) WHERE clock_out IS NULL;

-- ============================================================================

-- 3. payroll_records (احکام و رکوردهای فیش حقوقی ماهیانه پرداختی پرسنل)

-- ============================================================================

-- Replace uq_hr_payroll_period index in 01_hr_management_tables.sql

CREATE TABLE payroll_records (

    payroll_record_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    employee_id UUID NOT NULL REFERENCES employees(employee_id) ON DELETE RESTRICT,

    fiscal_period_id UUID NOT NULL,

    base_salary NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    allowances_total NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    deductions_total NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    tax_withheld NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    insurance_premium NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    net_payable NUMERIC(20,4) GENERATED ALWAYS AS (base_salary \+ allowances_total - deductions_total - tax_withheld - insurance_premium) STORED,

    is_disbursed BOOLEAN NOT NULL DEFAULT FALSE,

    disbursed_at TIMESTAMPTZ,

    journal_entry_id UUID, -- Logical Reference to accounting

    

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

-- Corrected Unique Index with Soft Delete

CREATE UNIQUE INDEX uq_hr_payroll_period 

    ON payroll_records(tenant_id, employee_id, fiscal_period_id) 

    WHERE deleted_at IS NULL;

CREATE INDEX idx_hr_payroll_ledger 

    ON payroll_records(tenant_id, fiscal_period_id) 

    WHERE deleted_at IS NULL;

-- ============================================================================

-- 4. hr_documents (اسناد، فرم‌ها و قراردادهای محرمانه منابع انسانی)

-- ============================================================================

CREATE TABLE hr_documents (

    hr_document_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    employee_id UUID NOT NULL REFERENCES employees(employee_id) ON DELETE RESTRICT,

    document_type_code VARCHAR(100) NOT NULL, -- e.g., 'CONTRACT', 'NDA', 'BACKGROUND_CHECK'

    document_title VARCHAR(200) NOT NULL,

    issue_date DATE,

    expiry_date DATE,

    attachment_id UUID, -- ارجاع منطقی به فریم‌ورک متمرکز پیوست فایل در لایه ۵ (attachments)

    status SMALLINT NOT NULL DEFAULT 1, -- 1: Active/Valid, 2: Expired, 3: Terminated

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ

);

CREATE INDEX idx_hr_documents_lookup ON hr_documents(tenant_id, employee_id) WHERE deleted_at IS NULL;

