# mfg quality and execution extension

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Vertical ERP Modules
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- ============================================================================

-- MODULE: MANUFACTURING QUALITY & EXECUTION EXTENSION LAYER

-- Database Engine: PostgreSQL 18 Enterprise Configuration

-- File Identity: mfg_quality_and_execution_extension.sql

-- Path: 05_Database_Design/ERP_Modules_Schema/07_Manufacturing_Production_Module

-- ============================================================================

CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- ============================================================================

-- 1. mfg_quality_inspections

-- مدیریت کنترل کیفیت ورودی، تولید و خروجی

-- ============================================================================

CREATE TABLE mfg_quality_inspections (

    inspection_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    inspection_type SMALLINT NOT NULL,

    -- 1: Incoming Material

    -- 2: Production Output

    -- 3: Final Product

    source_document_type VARCHAR(100),

    -- INV_DOCUMENT

    -- PRODUCTION_ORDER

    -- PURCHASE_RECEIPT

    source_document_id UUID,

    item_id UUID NOT NULL,

    -- Logical Reference -> inv_items

    batch_id UUID,

    -- Logical Reference -> inv_stock_batches

    inspection_number VARCHAR(100) NOT NULL,

    inspection_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    inspector_user_id UUID NOT NULL,

    sample_quantity NUMERIC(20,4) NOT NULL DEFAULT 0,

    accepted_quantity NUMERIC(20,4) NOT NULL DEFAULT 0,

    rejected_quantity NUMERIC(20,4) NOT NULL DEFAULT 0,

    qc_status SMALLINT NOT NULL DEFAULT 1,

    -- 1 Pending

    -- 2 Approved

    -- 3 Rejected

    -- 4 Quarantine

    notes VARCHAR(500),

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_mfg_quality_inspections_number

ON mfg_quality_inspections(tenant_id, inspection_number)

WHERE deleted_at IS NULL;

CREATE INDEX idx_mfg_quality_source

ON mfg_quality_inspections(source_document_type, source_document_id);

-- ============================================================================

-- 2. mfg_work_order_operations

-- اجرای واقعی عملیات تولید

-- ============================================================================

CREATE TABLE mfg_work_order_operations (

    operation_execution_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    production_order_id UUID NOT NULL

    REFERENCES mfg_production_orders(production_order_id)

    ON DELETE RESTRICT,

    routing_id UUID NOT NULL

    REFERENCES mfg_production_routing(routing_id)

    ON DELETE RESTRICT,

    work_center_id UUID NOT NULL

    REFERENCES mfg_work_centers(work_center_id)

    ON DELETE RESTRICT,

    operation_sequence INT NOT NULL,

    operator_user_id UUID,

    planned_start TIMESTAMPTZ,

    actual_start TIMESTAMPTZ,

    actual_end TIMESTAMPTZ,

    planned_hours NUMERIC(20,4) DEFAULT 0,

    actual_hours NUMERIC(20,4) DEFAULT 0,

    produced_quantity NUMERIC(20,4) DEFAULT 0,

    scrap_quantity NUMERIC(20,4) DEFAULT 0,

    operation_status SMALLINT NOT NULL DEFAULT 1,

    -- 1 Pending

    -- 2 Running

    -- 3 Completed

    -- 4 Stopped

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_mfg_work_operations_order

ON mfg_work_order_operations(production_order_id);

CREATE INDEX idx_mfg_work_operations_center

ON mfg_work_order_operations(work_center_id);

-- ============================================================================

-- 3. mfg_material_consumptions

-- مصرف واقعی مواد اولیه در تولید

-- ============================================================================

CREATE TABLE mfg_material_consumptions (

    consumption_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    production_order_id UUID NOT NULL

    REFERENCES mfg_production_orders(production_order_id)

    ON DELETE RESTRICT,

    bom_item_id UUID

    REFERENCES mfg_bom_items(bom_item_id)

    ON DELETE RESTRICT,

    material_item_id UUID NOT NULL,

    -- Logical Reference -> inv_items

    inventory_document_id UUID,

    -- Logical Reference -> inv_documents

    planned_quantity NUMERIC(20,4) NOT NULL DEFAULT 0,

    actual_quantity NUMERIC(20,4) NOT NULL DEFAULT 0,

    variance_quantity NUMERIC(20,4)

    GENERATED ALWAYS AS

    (actual_quantity - planned_quantity)

    STORED,

    consumption_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    consumed_by UUID,

    status SMALLINT NOT NULL DEFAULT 1,

    -- 1 Registered

    -- 2 Posted

    -- 3 Cancelled

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_mfg_material_consumption_order

ON mfg_material_consumptions(production_order_id);

CREATE INDEX idx_mfg_material_consumption_item

ON mfg_material_consumptions(material_item_id);

