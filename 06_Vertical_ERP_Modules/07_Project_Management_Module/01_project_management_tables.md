# 01 project management tables

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Vertical ERP Modules
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- ============================================================================

-- MODULE: PROJECT MANAGEMENT LAYER

-- Database Engine: PostgreSQL 18 Enterprise Configuration

-- File Identity: 01_project_management_tables.sql

-- Path: 06_Vertical_ERP_Modules/07_Project_Management_Module/

-- ============================================================================

CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- ============================================================================

-- 1. projects (تعریف، حاکمیت و چرخه عمر پروژه‌ها در سطح مستأجر)

-- ============================================================================

CREATE TABLE projects (

    project_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    project_code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    description TEXT,

    start_date DATE NOT NULL,

    end_date DATE,

    actual_end_date DATE,

    status SMALLINT NOT NULL DEFAULT 1, -- 1: Planning, 2: Active, 3: On Hold, 4: Completed, 5: Cancelled

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_projects_tenant_code ON projects(tenant_id, project_code) WHERE deleted_at IS NULL;

CREATE INDEX idx_projects_status_lookup ON projects(tenant_id, status) WHERE deleted_at IS NULL;

-- ============================================================================

-- 2. project_tasks (تسک‌ها، فعالیت‌ها و ساختار شکست کار WBS ذیل پروژه)

-- ============================================================================

CREATE TABLE project_tasks (

    task_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    project_id UUID NOT NULL REFERENCES projects(project_id) ON DELETE RESTRICT,

    parent_task_id UUID REFERENCES project_tasks(task_id) ON DELETE RESTRICT, -- ساختار درختی فعالیت‌ها

    wbs_code VARCHAR(50) NOT NULL, -- کد ساختار شکست کار (مثلا 1.1.2)

    title VARCHAR(200) NOT NULL,

    description TEXT,

    planned_start_date DATE NOT NULL,

    planned_end_date DATE NOT NULL,

    actual_start_date DATE,

    actual_end_date DATE,

    progress_percentage NUMERIC(5,2) NOT NULL DEFAULT 0.00, -- پیشرفت فیزیکی از ۰ تا ۱۰۰

    priority SMALLINT NOT NULL DEFAULT 2, -- 1: High, 2: Medium, 3: Low

    status SMALLINT NOT NULL DEFAULT 1, -- 1: Todo, 2: In Progress, 3: Review, 4: Done, 5: Blocked

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ

);

CREATE UNIQUE INDEX uq_project_tasks_wbs ON project_tasks(project_id, wbs_code) WHERE deleted_at IS NULL;

CREATE INDEX idx_project_tasks_hierarchy ON project_tasks(parent_task_id) WHERE deleted_at IS NULL;

-- ============================================================================

-- 3. project_members (اعضای پروژه و ماتریس مسئولیت‌ها و نقش‌های تیم کاری)

-- ============================================================================

-- Replace project_members table in 01_project_management_tables.sql

CREATE TABLE project_members (

    project_member_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    project_id UUID NOT NULL REFERENCES projects(project_id) ON DELETE RESTRICT,

    employee_id UUID NOT NULL, -- Logical Reference to hr.employees

    project_role VARCHAR(100) NOT NULL,

    joined_at DATE NOT NULL DEFAULT CURRENT_DATE,

    left_at DATE,

    is_active BOOLEAN NOT NULL DEFAULT TRUE,

    

    -- Standard Audit & Soft Delete Fields

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

-- Corrected Unique Index with Soft Delete Policy

CREATE UNIQUE INDEX uq_project_members_unique 

    ON project_members(project_id, employee_id) 

    WHERE deleted_at IS NULL;

CREATE INDEX idx_project_members_tenant 

    ON project_members(tenant_id) 

    WHERE deleted_at IS NULL;

-- ============================================================================

-- 4. resource_allocations (تخصیص منابع فیزیکی، ماشین‌آلات یا پرسنل انسانی به فعالیت‌ها)

-- ============================================================================

-- Replace resource_allocations table in 01_project_management_tables.sql

CREATE TABLE resource_allocations (

    allocation_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    task_id UUID NOT NULL REFERENCES project_tasks(task_id) ON DELETE RESTRICT,

    resource_type SMALLINT NOT NULL, -- 1: Human, 2: Equipment/Machine, 3: Material

    resource_id UUID NOT NULL, -- Polymorphic Logical Reference

    allocated_quantity NUMERIC(20,4) NOT NULL DEFAULT 1.0000, -- Standardized Precision

    start_date DATE NOT NULL,

    end_date DATE NOT NULL,

    

    -- Audit & Soft Delete Fields

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_resource_allocations_task 

    ON resource_allocations(task_id) 

    WHERE deleted_at IS NULL;

CREATE INDEX idx_resource_allocations_tenant 

    ON resource_allocations(tenant_id) 

    WHERE deleted_at IS NULL;

