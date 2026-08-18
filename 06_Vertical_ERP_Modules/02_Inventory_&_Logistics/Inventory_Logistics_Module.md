# Inventory Logistics Module

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Vertical ERP Modules
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- ============================================================================

-- MODULE: INVENTORY & DYNAMIC LOGISTICS ARCHITECTURE LAYER (PART 1 OF 2)

-- Database Engine: PostgreSQL 18 Enterprise Configuration

-- Path: 05_Database_Design/ERP_Modules_Schema/04_Inventory_Logistics_Module

-- ============================================================================

CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- 1. inv_warehouses (مدیریت انبارها و فضاهای فیزیکی هلدینگ)

CREATE TABLE inv_warehouses (

    warehouse_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL, -- قفل ایزوله‌سازی چندمستأجری

    branch_id UUID NOT NULL, -- ارجاع منطقی به لایه ۴ (ساختار سازمانی شعب)

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    is_bonded BOOLEAN NOT NULL DEFAULT FALSE,

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_inv_warehouses_code ON inv_warehouses(tenant_id, code) WHERE deleted_at IS NULL;

CREATE INDEX idx_inv_warehouses_tenant ON inv_warehouses(tenant_id) WHERE deleted_at IS NULL;

-- 2. inv_locations (ساختار درختی قفسه‌ها، ردیف‌ها و زون‌های داخلی هر انبار)

CREATE TABLE inv_locations (

    location_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    warehouse_id UUID NOT NULL REFERENCES inv_warehouses(warehouse_id) ON DELETE RESTRICT,

    parent_location_id UUID REFERENCES inv_locations(location_id) ON DELETE RESTRICT,

    tenant_id UUID NOT NULL,

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    aisle VARCHAR(50), -- ردیف

    rack VARCHAR(50),  -- قفسه

    shelf VARCHAR(50), -- طبقه

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_inv_locations_code ON inv_locations(warehouse_id, code) WHERE deleted_at IS NULL;

CREATE INDEX idx_inv_locations_parent ON inv_locations(parent_location_id) WHERE deleted_at IS NULL;

-- 3. inv_items (کاتالوگ مرکزی مشخصات کالاها و خدمات سازمان)

CREATE TABLE inv_items (

    item_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    item_group_id UUID NOT NULL, -- ارجاع منطقی به کاتالوگ داده‌های پایه فاز ۱

    uom_id UUID NOT NULL,        -- ارجاع منطقی به واحدهای سنجش فاز ۱

    code VARCHAR(100) NOT NULL,

    name VARCHAR(300) NOT NULL,

    description VARCHAR(500),

    item_type SMALLINT NOT NULL DEFAULT 1, -- (1: Stockable, 2: Service, 3: Expense)

    valuation_method SMALLINT NOT NULL DEFAULT 1, -- (1: FIFO, 2: Moving Average)

    extra_attributes JSONB, -- لایه پویای مشخصات کالا (رنگ، سایز، متادیتای شیمیایی/شبکه‌ای)

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_inv_items_code ON inv_items(tenant_id, code) WHERE deleted_at IS NULL;

CREATE INDEX idx_inv_items_extra_attributes ON inv_items USING GIN (extra_attributes); -- ایندکس پویا روی متادیتای JSONB

-- 4. inv_item_barcodes (بارکدها، شماره سریال‌ها و SKUs فرعی متصل به کالا)

CREATE TABLE inv_item_barcodes (

    barcode_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    item_id UUID NOT NULL REFERENCES inv_items(item_id) ON DELETE RESTRICT,

    tenant_id UUID NOT NULL,

    barcode VARCHAR(100) NOT NULL,

    barcode_type VARCHAR(50) NOT NULL DEFAULT 'EAN13',

    sku VARCHAR(100),

    is_primary BOOLEAN NOT NULL DEFAULT FALSE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    deleted_at TIMESTAMPTZ

);

CREATE UNIQUE INDEX uq_inv_item_barcodes ON inv_item_barcodes(tenant_id, barcode) WHERE deleted_at IS NULL;

CREATE INDEX idx_inv_item_barcodes_lookup ON inv_item_barcodes(item_id) WHERE deleted_at IS NULL;

-- ============================================================================

-- MODULE: INVENTORY & DYNAMIC LOGISTICS ARCHITECTURE LAYER (PART 2 OF 2)

-- Database Engine: PostgreSQL 18 Enterprise Configuration

-- Path: 05_Database_Design/ERP_Modules_Schema/04_Inventory_Logistics_Module

-- ============================================================================

-- 5. inv_stock_balances (لجر زنده موجودی کالا به تفکیک قفسه و انبار)

CREATE TABLE inv_stock_balances (

    stock_balance_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    warehouse_id UUID NOT NULL REFERENCES inv_warehouses(warehouse_id) ON DELETE RESTRICT,

    location_id UUID NOT NULL REFERENCES inv_locations(location_id) ON DELETE RESTRICT,

    item_id UUID NOT NULL REFERENCES inv_items(item_id) ON DELETE RESTRICT,

    quantity_on_hand NUMERIC(20,4) NOT NULL DEFAULT 0.0000, -- موجودی فیزیکی واقعی

    quantity_reserved NUMERIC(20,4) NOT NULL DEFAULT 0.0000, -- موجودی رزرو شده در جریان گردش کار

    quantity_available NUMERIC(20,4) GENERATED ALWAYS AS (quantity_on_hand - quantity_reserved) STORED, -- محاسبه خودکار سرعت بالا

    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_inv_stock_balances ON inv_stock_balances(location_id, item_id);

CREATE INDEX idx_inv_stock_balances_tenant_lookup ON inv_stock_balances(tenant_id, warehouse_id, item_id);

-- 6. inv_documents (سربرگ اسناد تراکنشی انبار - حواله خروج، رسید ورود، انتقال و انبارگردانی)

CREATE TABLE inv_documents (

    document_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    fiscal_period_id UUID NOT NULL, -- ارجاع منطقی به دوره مالی فاز ۱

    document_type SMALLINT NOT NULL, -- (1: Goods Receipt, 2: Goods Issue, 3: Inventory Transfer, 4: Cycle Adjustment)

    document_number VARCHAR(100) NOT NULL,

    posting_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    source_document_type VARCHAR(100), -- رفرنس متنی پویا به لایه‌های دیگر (مثل 'SALES_INVOICE') بر اساس اصل عدم جفت‌شدگی

    source_document_id UUID,          -- شناسه منطقی سند مرجع بدون کلید خارجی فیزیکی

    business_partner_id UUID,         -- ارجاع منطقی به شریک تجاری فاز ۱ (تامین‌کننده/مشتری)

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Draft, 2: Pending Approval, 3: Posted/Released, 4: Voided)

    description VARCHAR(500),

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID

);

CREATE UNIQUE INDEX uq_inv_documents_number ON inv_documents(tenant_id, document_number) WHERE deleted_at IS NULL;

CREATE INDEX idx_inv_documents_cross_lookup ON inv_documents(source_document_id, source_document_type);

-- 7. inv_document_items (اقلام و سطور اسناد ورود و خروج انبار)

CREATE TABLE inv_document_items (

    document_item_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    document_id UUID NOT NULL REFERENCES inv_documents(document_id) ON DELETE RESTRICT,

    tenant_id UUID NOT NULL,

    item_id UUID NOT NULL REFERENCES inv_items(item_id) ON DELETE RESTRICT,

    from_location_id UUID REFERENCES inv_locations(location_id) ON DELETE RESTRICT, -- مبدا (برای حواله‌ها و انتقال)

    to_location_id UUID REFERENCES inv_locations(location_id) ON DELETE RESTRICT,   -- مقصد (برای رسیدها و انتقال)

    batch_number VARCHAR(100), -- ارجاع منطقی اختیاری به جدول دسته‌های تولیدی

    quantity NUMERIC(20,4) NOT NULL,

    unit_cost NUMERIC(20,4) NOT NULL DEFAULT 0.0000, -- قیمت واحد مبنای ارزش‌گذاری انبار

    total_cost NUMERIC(20,4) GENERATED ALWAYS AS (quantity * unit_cost) STORED, -- محاسبه خودکار دیتابیسی

    sort_order INT NOT NULL DEFAULT 0,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_inv_document_items_parent ON inv_document_items(document_id);

-- 8. inv_stock_batches (مدیریت دسته‌های تولیدی، تاریخ انقضا و قرنطینه کیفی کالاها)

CREATE TABLE inv_stock_batches (

    batch_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    item_id UUID NOT NULL REFERENCES inv_items(item_id) ON DELETE RESTRICT,

    batch_number VARCHAR(100) NOT NULL,

    quantity_produced NUMERIC(20,4) NOT NULL,

    quantity_remaining NUMERIC(20,4) NOT NULL,

    production_date DATE,

    expiration_date DATE,

    qc_status SMALLINT NOT NULL DEFAULT 1, -- (1: Pending Inspection, 2: Approved/Released, 3: Quarantined/Locked)

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_inv_stock_batches ON inv_stock_batches(tenant_id, item_id, batch_number) WHERE deleted_at IS NULL;

CREATE INDEX idx_inv_stock_batches_expiration ON inv_stock_batches(expiration_date) WHERE qc_status = 1;

