# mfg production table definitions

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Vertical ERP Modules
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- ============================================================================

-- MODULE: MANUFACTURING & PRODUCTION ARCHITECTURE LAYER (PART 1 OF 2)

-- Database Engine: PostgreSQL 18 Enterprise Configuration

-- File Identity: mfg_production_table_definitions.sql

-- Path: 05_Database_Design/ERP_Modules_Schema/07_Manufacturing_Production_Module

-- ============================================================================

CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- 1. mfg_work_centers (ایستگاه‌های کاری، ماشین‌آلات و خطوط تولید کارخانه)

CREATE TABLE mfg_work_centers (

    work_center_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL, -- ایزوله‌سازی چندمستأجری

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    capacity_hours_per_day NUMERIC(20,4) NOT NULL DEFAULT 8.0000, -- پایش ظرفیت با دقت بالا

    efficiency_percentage NUMERIC(20,4) NOT NULL DEFAULT 100.0000,

    cost_per_hour NUMERIC(20,4) NOT NULL DEFAULT 0.0000, -- بهای تمام‌شده ساعتی ماشین

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_mfg_work_centers_code ON mfg_work_centers(tenant_id, code) WHERE deleted_at IS NULL;

-- 2. mfg_boms (سربرگ درختواره مواد و فرمولاسیون کالاها - Bill of Materials)

CREATE TABLE mfg_boms (

    bom_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    item_id UUID NOT NULL, -- ارجاع منطقی به کاتالوگ کالای ماژول انبار

    bom_code VARCHAR(100) NOT NULL,

    version_number INT NOT NULL DEFAULT 1,

    is_active BOOLEAN NOT NULL DEFAULT TRUE,

    batch_size NUMERIC(20,4) NOT NULL DEFAULT 1.0000, -- مبنای محاسباتی فرمول (مثلا فرمول به ازای ۱۰۰ کیلوگرم خروجی)

    description VARCHAR(500),

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Draft/Pending QC, 2: Approved/Released, 3: Obsolete)

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_mfg_boms_code_version ON mfg_boms(tenant_id, bom_code, version_number) WHERE deleted_at IS NULL;

CREATE INDEX idx_mfg_boms_item ON mfg_boms(tenant_id, item_id) WHERE deleted_at IS NULL;

-- 3. mfg_bom_items (ریز مواد اولیه و قطعات ثانویه مورد نیاز در هر فرمول ساخت)

CREATE TABLE mfg_bom_items (

    bom_item_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    bom_id UUID NOT NULL REFERENCES mfg_boms(bom_id) ON DELETE RESTRICT, -- قفل سخت‌گیرانه مهندسی ساخت

    tenant_id UUID NOT NULL,

    component_item_id UUID NOT NULL, -- ارجاع منطقی به ماده اولیه مصرفی در انبار

    uom_id UUID NOT NULL,            -- ارجاع منطقی به واحد سنجش فاز ۱

    quantity NUMERIC(20,4) NOT NULL,  -- مقدار مورد نیاز استاندارد

    scrap_percentage NUMERIC(20,4) NOT NULL DEFAULT 0.0000, -- درصد مجاز پرت و ضایعات کالا

    sort_order INT NOT NULL DEFAULT 0,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_mfg_bom_items_parent ON mfg_bom_items(bom_id);

-- ============================================================================

-- MODULE: MANUFACTURING & PRODUCTION ARCHITECTURE LAYER (PART 2 OF 2)

-- Database Engine: PostgreSQL 18 Enterprise Configuration

-- File Identity: mfg_production_table_definitions.sql

-- Path: 05_Database_Design/ERP_Modules_Schema/07_Manufacturing_Production_Module

-- ============================================================================

-- 4. mfg_production_orders (دستورات و سفارشات قطعی تولید زنده کارخانه)

CREATE TABLE mfg_production_orders (

    production_order_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    bom_id UUID NOT NULL REFERENCES mfg_boms(bom_id) ON DELETE RESTRICT,

    order_number VARCHAR(100) NOT NULL,

    target_item_id UUID NOT NULL, -- ارجاع منطقی به محصول نهایی هدف در انبار

    quantity_to_produce NUMERIC(20,4) NOT NULL,

    quantity_produced NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    start_date DATE NOT NULL,

    end_date DATE,

    source_document_type VARCHAR(50), -- رفرنس متنی به لایه‌های دیگر (مانند 'SAL_ORDER' یا 'MRP')

    source_document_id UUID,          -- شناسه منطقی سند مرجع بدون کلید خارجی فیزیکی

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Planned, 2: Released/In Progress, 3: Completed, 4: Cancelled)

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_mfg_prod_orders_num ON mfg_production_orders(tenant_id, order_number) WHERE deleted_at IS NULL;

CREATE INDEX idx_mfg_prod_orders_source ON mfg_production_orders(source_document_id, source_document_type) WHERE deleted_at IS NULL;

-- 5. mfg_production_routing (توالی و مراحل گام‌های عملیاتی ساخت در ایستگاه‌ها)

CREATE TABLE mfg_production_routing (

    routing_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    production_order_id UUID NOT NULL REFERENCES mfg_production_orders(production_order_id) ON DELETE RESTRICT,

    tenant_id UUID NOT NULL,

    work_center_id UUID NOT NULL REFERENCES mfg_work_centers(work_center_id) ON DELETE RESTRICT,

    operation_sequence INT NOT NULL, -- (e.g., 10: Cutting, 20: Welding, 30: Assembly)

    operation_name VARCHAR(150) NOT NULL,

    standard_setup_time_hours NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    standard_run_time_hours NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Pending, 2: Active, 3: Completed)

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

-- Replace uq_mfg_routing_sequence in mfg_production_table_definitions.sql

CREATE UNIQUE INDEX uq_mfg_routing_sequence 

    ON mfg_production_routing(production_order_id, operation_sequence) 

    WHERE deleted_at IS NULL;

-- 6. mfg_production_logs (ثبت زنده مصرف واقعی مواد و کارکرد زمانی پای خط تولید)

-- Replace mfg_production_logs definition in mfg_production_table_definitions.sql

CREATE TABLE mfg_production_logs (

    production_log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    production_order_id UUID NOT NULL REFERENCES mfg_production_orders(production_order_id) ON DELETE RESTRICT,

    routing_id UUID REFERENCES mfg_production_routing(routing_id) ON DELETE RESTRICT,

    log_type SMALLINT NOT NULL, -- 1: Material Consumption, 2: Labor/Machine Time, 3: Scrap

    item_id UUID, -- Logical Reference to inv_items

    quantity_consumed NUMERIC(20,4) NOT NULL DEFAULT 0.0000, -- Fixed precision to NUMERIC(20,4)

    hours_spent NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    logged_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

