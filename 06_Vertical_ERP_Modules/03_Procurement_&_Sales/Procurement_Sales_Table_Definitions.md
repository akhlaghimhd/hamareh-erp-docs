# Procurement Sales Table Definitions

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Vertical ERP Modules
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- ============================================================================

-- MODULE: PROCUREMENT & SUPPLY CHAIN ARCHITECTURE LAYER (PART 1 OF 2)

-- Database Engine: PostgreSQL 18 Enterprise Configuration

-- File Identity: proc_sales_table_definitions.sql

-- Path: 05_Database_Design/ERP_Modules_Schema/05_Procurement_Sales_Module

-- ============================================================================

CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- 1. pur_requisitions (درخواست‌های خرید داخلی دپارتمان‌ها)

CREATE TABLE pur_requisitions (

    requisition_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    department_id UUID NOT NULL, -- ارجاع منطقی به لایه ۴ سازمانی

    requisition_number VARCHAR(100) NOT NULL,

    required_date DATE NOT NULL,

    priority SMALLINT NOT NULL DEFAULT 2, -- (1: High, 2: Medium, 3: Low)

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Draft, 2: Pending, 3: Approved, 4: Rejected)

    description VARCHAR(500),

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_pur_requisitions_num ON pur_requisitions(tenant_id, requisition_number) WHERE deleted_at IS NULL;

-- 2. pur_orders (سفارشات خرید قطعی به تأمین‌کننده)

CREATE TABLE pur_orders (

    purchase_order_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    fiscal_period_id UUID NOT NULL, -- ارجاع منطقی دوره مالی فاز ۱

    business_partner_id UUID NOT NULL, -- ارجاع منطقی به تامین‌کننده در داده‌های پایه

    payment_term_id UUID REFERENCES payment_terms(payment_term_id) ON DELETE RESTRICT,

    shipment_method_id UUID REFERENCES shipment_methods(shipment_method_id) ON DELETE RESTRICT,

    po_number VARCHAR(100) NOT NULL,

    order_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    currency_id UUID NOT NULL, -- ارجاع منطقی به کاتالوگ ارزها

    exchange_rate NUMERIC(20,4) NOT NULL DEFAULT 1.0000,

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Draft, 2: Confirmed, 3: Closed, 4: Cancelled)

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_pur_orders_num ON pur_orders(tenant_id, po_number) WHERE deleted_at IS NULL;

CREATE INDEX idx_pur_orders_partner ON pur_orders(tenant_id, business_partner_id) WHERE deleted_at IS NULL;

-- 3. pur_invoices (فاکتورهای خرید رسمی دریافتی)

CREATE TABLE pur_invoices (

    purchase_invoice_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    fiscal_period_id UUID NOT NULL,

    business_partner_id UUID NOT NULL,

    purchase_order_id UUID REFERENCES pur_orders(purchase_order_id) ON DELETE RESTRICT,

    invoice_number VARCHAR(100) NOT NULL,

    posting_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    due_date DATE NOT NULL,

    total_lines_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    total_discount_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    total_tax_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    payable_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000, -- مبلغ نهایی فاکتور خرید

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Draft, 2: Open/Unpaid, 3: Paid, 4: Voided)

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_pur_invoices_num ON pur_invoices(tenant_id, invoice_number) WHERE deleted_at IS NULL;

-- 4. sal_quotations (پیش‌فاکتورهای فروش صادر شده برای مشتریان)

CREATE TABLE sal_quotations (

    quotation_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    business_partner_id UUID NOT NULL, -- ارجاع منطقی به مشتری

    quotation_number VARCHAR(100) NOT NULL,

    issue_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    expiry_date DATE NOT NULL,

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Draft, 2: Sent, 3: Accepted, 4: Expired)

    description VARCHAR(500),

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_sal_quotations_num ON sal_quotations(tenant_id, quotation_number) WHERE deleted_at IS NULL;

-- ============================================================================

-- MODULE: PROCUREMENT & SUPPLY CHAIN ARCHITECTURE LAYER (PART 2 OF 2)

-- Database Engine: PostgreSQL 18 Enterprise Configuration

-- File Identity: proc_sales_table_definitions.sql

-- Path: 05_Database_Design/ERP_Modules_Schema/05_Procurement_Sales_Module

-- ============================================================================

-- 5. sal_orders (سفارشات فروش قطعی پس از اعتبارسنجی پارتنر)

CREATE TABLE sal_orders (

    sales_order_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    fiscal_period_id UUID NOT NULL,

    business_partner_id UUID NOT NULL,

    quotation_id UUID REFERENCES sal_quotations(quotation_id) ON DELETE RESTRICT,

    payment_term_id UUID REFERENCES payment_terms(payment_term_id) ON DELETE RESTRICT,

    shipment_method_id UUID REFERENCES shipment_methods(shipment_method_id) ON DELETE RESTRICT,

    so_number VARCHAR(100) NOT NULL,

    order_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    currency_id UUID NOT NULL,

    exchange_rate NUMERIC(20,4) NOT NULL DEFAULT 1.0000,

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Draft, 2: Approved, 3: Processing, 4: Closed, 5: Cancelled)

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_sal_orders_num ON sal_orders(tenant_id, so_number) WHERE deleted_at IS NULL;

CREATE INDEX idx_sal_orders_partner ON sal_orders(tenant_id, business_partner_id) WHERE deleted_at IS NULL;

-- 6. sal_invoices (فاکتورهای فروش رسمی و قانونی مستأجران)

CREATE TABLE sal_invoices (

    sales_invoice_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    fiscal_period_id UUID NOT NULL,

    business_partner_id UUID NOT NULL,

    sales_order_id UUID REFERENCES sal_orders(sales_order_id) ON DELETE RESTRICT,

    invoice_number VARCHAR(100) NOT NULL,

    posting_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    due_date DATE NOT NULL,

    total_lines_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    total_discount_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    total_tax_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    receivable_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000, -- مبنای کنترل سقف اعتبار مشتری

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Draft, 2: Open/Unpaid, 3: Partially Paid, 4: Fully Paid, 5: Voided)

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_sal_invoices_num ON sal_invoices(tenant_id, invoice_number) WHERE deleted_at IS NULL;

-- 7. trade_document_items (موتور متمرکز و یکپارچه اقلام و سطور تمام اسناد خرید و فروش بالا)

CREATE TABLE trade_document_items (

    document_item_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    source_document_type VARCHAR(50) NOT NULL, -- (e.g., 'PUR_ORDER', 'PUR_INVOICE', 'SAL_ORDER', 'SAL_INVOICE')

    source_document_id UUID NOT NULL,          -- ارجاع پویای منطقی چندشکلی بدون FK فیزیکی بر اساس اصول تفکیک دامنه

    item_id UUID NOT NULL,                     -- ارجاع منطقی به کاتالوگ کالای ماژول انبار

    uom_id UUID NOT NULL,                      -- ارجاع منطقی به واحدهای سنجش فاز ۱

    quantity NUMERIC(20,4) NOT NULL,

    unit_price NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    gross_amount NUMERIC(20,4) GENERATED ALWAYS AS (quantity * unit_price) STORED,

    discount_percentage NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    discount_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    tax_category_id UUID,                      -- ارجاع منطقی به قوانین مالیاتی فاز ۱

    tax_percentage NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    tax_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    net_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000, -- (Gross - Discount \+ Tax) محاسبه نهایی در بک‌اند

    sort_order INT NOT NULL DEFAULT 0,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_trade_items_polymorphic ON trade_document_items(source_document_type, source_document_id);

CREATE INDEX idx_trade_items_tenant ON trade_document_items(tenant_id);

-- 8. fin_payment_schedules (برنامه‌ریزی، اقساط و سررسید تسویه فاکتورها)

CREATE TABLE fin_payment_schedules (

    payment_schedule_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    source_document_type VARCHAR(50) NOT NULL, -- (e.g., 'PUR_INVOICE', 'SAL_INVOICE')

    source_document_id UUID NOT NULL,

    due_date DATE NOT NULL,

    expected_amount NUMERIC(20,4) NOT NULL,

    paid_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Pending, 2: Partially Paid, 3: Settled, 4: Overdue)

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_fin_schedules_lookup ON fin_payment_schedules(source_document_type, source_document_id);

CREATE INDEX idx_fin_schedules_due ON fin_payment_schedules(tenant_id, due_date) WHERE status IN (1, 2);

-- 9. fin_cash_transactions (دفتر ثبت پولی ورود و خروج مستقیم خزانه‌داری مقدماتی)

CREATE TABLE fin_cash_transactions (

    cash_transaction_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    payment_schedule_id UUID REFERENCES fin_payment_schedules(payment_schedule_id) ON DELETE RESTRICT,

    bank_account_id UUID NOT NULL, -- ارجاع منطقی به حساب‌های بانکی فاز ۱

    transaction_type SMALLINT NOT NULL, -- (1: Receipt/Varyez, 2: Payment/Bardasht)

    amount NUMERIC(20,4) NOT NULL,

    payment_reference VARCHAR(150), -- شماره فیش یا پیگیری بانکی

    transaction_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Pending Cleared, 2: Cleared/Taeid Shodeh, 3: Rejected)

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ

);

CREATE INDEX idx_fin_cash_tx_tenant ON fin_cash_transactions(tenant_id);

CREATE INDEX idx_fin_cash_tx_bank ON fin_cash_transactions(bank_account_id);

-- ============================================================================

-- 1. purchase_receipts (رسید انبار / رسید فیزیکی کالاها از تامین‌کنندگان)

-- ============================================================================

CREATE TABLE purchase_receipts (

    receipt_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    receipt_number VARCHAR(100) NOT NULL,

    id_purchase_order_source UUID, -- ارجاع منطقی به سفارش خرید بالادستی

    supplier_id UUID NOT NULL, -- ارجاع منطقی به business_partners

    warehouse_id UUID NOT NULL, -- ارجاع منطقی به inv_warehouses

    receipt_date DATE NOT NULL,

    status SMALLINT NOT NULL DEFAULT 1, -- 1: Draft, 2: Inspected/Pending Putaway, 3: Completed

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_pur_receipts_num ON purchase_receipts(tenant_id, receipt_number) WHERE deleted_at IS NULL;

-- ============================================================================

-- 2. sales_delivery_orders (حواله خروج / دستور فیزیکی تحویل کالا به مشتریان)

-- ============================================================================

CREATE TABLE sales_delivery_orders (

    delivery_order_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    delivery_number VARCHAR(100) NOT NULL,

    id_sales_order_source UUID, -- ارجاع منطقی به سفارش فروش بالادستی

    customer_id UUID NOT NULL, -- ارجاع منطقی به business_partners

    warehouse_id UUID NOT NULL, -- ارجاع منطقی به inv_warehouses

    delivery_date DATE NOT NULL,

    status SMALLINT NOT NULL DEFAULT 1, -- 1: Planned, 2: Picking, 3: Shipped/Completed

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_sal_deliveries_num ON sales_delivery_orders(tenant_id, delivery_number) WHERE deleted_at IS NULL;

-- ============================================================================

-- 3. return_orders (سربرگ اسناد برگشت از خرید یا برگشت از فروش)

-- ============================================================================

CREATE TABLE return_orders (

    return_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    return_number VARCHAR(100) NOT NULL,

    return_type SMALLINT NOT NULL, -- 1: Return from Purchase (برگشت از خرید), 2: Return from Sales (برگشت از فروش)

    source_document_type VARCHAR(100), -- PUR_INVOICE, SAL_INVOICE, PUR_RECEIPT, SAL_DELIVERY

    source_document_id UUID, -- ارجاع منطقی به سند اصلی

    business_partner_id UUID NOT NULL, -- تامین‌کننده یا مشتری

    return_date DATE NOT NULL,

    reason_code VARCHAR(50),

    status SMALLINT NOT NULL DEFAULT 1, -- 1: Draft, 2: Approved, 3: Processed

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_return_orders_num ON return_orders(tenant_id, return_number) WHERE deleted_at IS NULL;

-- ============================================================================

-- 4. return_order_items (سطور و ریز اقلام کالاهای مرجوعی)

-- ============================================================================

CREATE TABLE return_order_items (

    return_item_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    return_id UUID NOT NULL REFERENCES return_orders(return_id) ON DELETE RESTRICT,

    tenant_id UUID NOT NULL,

    item_id UUID NOT NULL, -- ارجاع منطقی به کاتالوگ کالا

    quantity NUMERIC(20,4) NOT NULL,

    unit_price NUMERIC(20,4) NOT NULL,

    tax_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    net_amount NUMERIC(20,4) NOT NULL,

    sort_order INT NOT NULL DEFAULT 0

);

CREATE INDEX idx_return_items_parent ON return_order_items(return_id);

-- ۱. اصلاح جدول سفارشات خرید

ALTER TABLE pur_orders 

ADD COLUMN supplier_reference VARCHAR(100),

ADD COLUMN approval_status SMALLINT NOT NULL DEFAULT 1; -- 1: Draft, 2: Pending, 3: Approved, 4: Rejected

-- ۲. اصلاح جدول سفارشات فروش

ALTER TABLE sal_orders 

ADD COLUMN delivery_date DATE,

ADD COLUMN approval_status SMALLINT NOT NULL DEFAULT 1;

-- ۳. اصلاح فاکتورهای خرید (افزودن شماره فاکتور قانونی/مالیاتی سیستم رسمی کشور)

ALTER TABLE pur_invoices 

ADD COLUMN tax_invoice_number VARCHAR(100);

-- ۴. اصلاح فاکتورهای فروش

ALTER TABLE sal_invoices 

ADD COLUMN tax_invoice_number VARCHAR(100);

-- ۵. اصلاح جداول خزانه و تسویه جهت پشتیبانی از چندارزی

ALTER TABLE fin_payment_schedules ADD COLUMN currency_id UUID;

ALTER TABLE fin_cash_transactions ADD COLUMN currency_id UUID;

