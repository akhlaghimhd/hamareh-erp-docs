# fin accounting table definitions

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Vertical ERP Modules
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- ============================================================================

-- MODULE: FINANCIAL ACCOUNTING & GENERAL LEDGER LAYER (PART 1 OF 2)

-- Database Engine: PostgreSQL 18 Enterprise Configuration

-- File Identity: fin_accounting_table_definitions.sql

-- Path: 05_Database_Design/ERP_Modules_Schema/06_Financial_Accounting_Module

-- ============================================================================

CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- 1. fin_acc_accounts (درختواره کدینگ حسابداری - کل و معین)

CREATE TABLE fin_acc_accounts (

    account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL, -- ایزوله‌سازی چندمستأجری

    parent_account_id UUID REFERENCES fin_acc_accounts(account_id) ON DELETE RESTRICT,

    account_code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    account_type SMALLINT NOT NULL, -- (1: Asset, 2: Liability, 3: Equity, 4: Revenue, 5: Expense)

    account_level SMALLINT NOT NULL DEFAULT 1, -- (1: Kol, 2: Moein)

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_fin_acc_accounts_code ON fin_acc_accounts(tenant_id, account_code) WHERE deleted_at IS NULL;

CREATE INDEX idx_fin_acc_accounts_parent ON fin_acc_accounts(parent_account_id) WHERE deleted_at IS NULL;

-- 2. fin_acc_polymorphic_details (حساب‌های تفصیلی شناور و چندشکلی)

CREATE TABLE fin_acc_polymorphic_details (

    detail_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    detail_code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    sub_ledger_type VARCHAR(100) NOT NULL, -- (e.g., 'BUSINESS_PARTNER', 'BANK_ACCOUNT', 'USER_PROFILE')

    sub_ledger_id UUID NOT NULL,          -- رفرنس منطقی پویا بدون FK فیزیکی

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ

);

CREATE UNIQUE INDEX uq_fin_acc_details_code ON fin_acc_polymorphic_details(tenant_id, detail_code) WHERE deleted_at IS NULL;

CREATE INDEX idx_fin_acc_details_polymorphic ON fin_acc_polymorphic_details(sub_ledger_type, sub_ledger_id) WHERE deleted_at IS NULL;

-- 3. fin_acc_cost_centers (مراکز هزینه و مراکز سودآوری پروژه)

CREATE TABLE fin_acc_cost_centers (

    cost_center_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    parent_id UUID REFERENCES fin_acc_cost_centers(cost_center_id) ON DELETE RESTRICT,

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    center_type SMALLINT NOT NULL DEFAULT 1, -- (1: Cost Center, 2: Profit Center)

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_fin_acc_cost_centers_code ON fin_acc_cost_centers(tenant_id, code) WHERE deleted_at IS NULL;

-- ============================================================================

-- MODULE: FINANCIAL ACCOUNTING & GENERAL LEDGER LAYER (PART 2 OF 2)

-- Database Engine: PostgreSQL 18 Enterprise Configuration

-- File Identity: fin_accounting_table_definitions.sql

-- Path: 05_Database_Design/ERP_Modules_Schema/06_Financial_Accounting_Module

-- ============================================================================

-- 4. fin_acc_journal_entries (سربرگ و هدر اسناد حسابداری)

CREATE TABLE fin_acc_journal_entries (

    journal_entry_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    fiscal_period_id UUID NOT NULL, -- ارجاع منطقی دوره مالی فاز ۱

    company_id UUID NOT NULL,       -- ارجاع منطقی به لایه ۴ ساختار سازمانی

    entry_number VARCHAR(100) NOT NULL,

    document_date DATE NOT NULL,

    posting_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    source_document_type VARCHAR(100), -- رفرنس متنی پویا به لایه‌های دیگر (مثل 'SAL_INVOICE' یا 'INV_DOCUMENT')

    source_document_id UUID,          -- شناسه منطقی سند مرجع بدون کلید خارجی فیزیکی

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Draft, 2: Pending Approval, 3: Posted/Released, 4: Voided)

    description VARCHAR(500),

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_fin_acc_entries_num ON fin_acc_journal_entries(tenant_id, company_id, entry_number) WHERE deleted_at IS NULL;

CREATE INDEX idx_fin_acc_entries_source ON fin_acc_journal_entries(source_document_id, source_document_type) WHERE deleted_at IS NULL;

-- 5. fin_acc_journal_items (سطور و آرتیکل‌های تراکنشی سند حسابداری - Double-Entry)

-- اصلاح محاسباتی: ارتقا به NUMERIC(20,4) برای مبالغ ارزی و پایه جهت پایش دقیق تراز مالی

CREATE TABLE fin_acc_journal_items (

    journal_item_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    journal_entry_id UUID NOT NULL REFERENCES fin_acc_journal_entries(journal_entry_id) ON DELETE RESTRICT,

    tenant_id UUID NOT NULL,

    account_id UUID NOT NULL REFERENCES fin_acc_accounts(account_id) ON DELETE RESTRICT,

    detail_id UUID REFERENCES fin_acc_polymorphic_details(detail_id) ON DELETE RESTRICT, -- اتصال به تفصیلی شناور

    cost_center_id UUID REFERENCES fin_acc_cost_centers(cost_center_id) ON DELETE RESTRICT, -- اتصال به مرکز هزینه

    debit_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    credit_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    currency_id UUID NOT NULL, -- ارجاع منطقی به کاتالوگ عمومی ارزها

    exchange_rate NUMERIC(20,4) NOT NULL DEFAULT 1.0000,

    source_currency_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000, -- مبلغ واقعی به ارز محلی فاکتور

    description VARCHAR(500),

    sort_order INT NOT NULL DEFAULT 0,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()

);

CREATE INDEX idx_fin_acc_items_entry ON fin_acc_journal_items(journal_entry_id);

CREATE INDEX idx_fin_acc_items_account_lookup ON fin_acc_journal_items(tenant_id, account_id, detail_id);

-- 6. fin_acc_exchange_rates (جدول لاگ روزانه نرخ برابری ارزها)

CREATE TABLE fin_acc_exchange_rates (

    exchange_rate_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    from_currency_id UUID NOT NULL, -- ارجاع منطقی ارز فرعی

    to_currency_id UUID NOT NULL,   -- ارجاع منطقی ارز پایه سازمان

    rate_date DATE NOT NULL,

    rate_value NUMERIC(20,4) NOT NULL, -- نرخ تبدیل با دقت ۴ رقم اعشار

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    deleted_at TIMESTAMPTZ

);

CREATE UNIQUE INDEX uq_fin_acc_rates_date ON fin_acc_exchange_rates(tenant_id, from_currency_id, to_currency_id, rate_date) WHERE deleted_at IS NULL;

-- 7. fin_acc_closing_logs (لاگ حاکمیتی فرآیندهای حساس بستن ماهیانه/سالانه دفاتر مالی)

CREATE TABLE fin_acc_closing_logs (

    closing_log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    fiscal_period_id UUID NOT NULL, -- ارجاع منطقی به دوره مالی فاز ۱

    closing_type SMALLINT NOT NULL, -- (1: Monthly Close, 2: Year-End Preliminary, 3: Year-End Permanent/Ikhtitami)

    closing_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    closing_user_id UUID NOT NULL, -- Logical Reference to Layer 4 users

    closing_entry_id UUID REFERENCES fin_acc_journal_entries(journal_entry_id) ON DELETE RESTRICT, -- سند اختتامیه صادر شده

    status SMALLINT NOT NULL DEFAULT 1, -- (1: Successful, 2: Reverted)

    description VARCHAR(500),

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ

);

CREATE INDEX idx_fin_acc_closing_period ON fin_acc_closing_logs(tenant_id, fiscal_period_id);

-- ============================================================================

-- 1. fin_acc_tax_transactions (لجر مستقل تراکنش‌های مالیاتی و ارزش افزوده)

-- ============================================================================

CREATE TABLE fin_acc_tax_transactions (

    tax_transaction_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    source_document_type VARCHAR(100) NOT NULL, -- PUR_INVOICE, SAL_INVOICE

    source_document_id UUID NOT NULL, -- ارجاع منطقی ناهمگام

    tax_category_id UUID NOT NULL, -- ارجاع منطقی به لایه ۵ کاتالوگ مالیاتی

    taxable_amount NUMERIC(20,4) NOT NULL,

    tax_rate NUMERIC(20,4) NOT NULL,

    tax_amount NUMERIC(20,4) NOT NULL,

    transaction_date DATE NOT NULL,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()

);

CREATE INDEX idx_fin_tax_lookup ON fin_acc_tax_transactions(tenant_id, source_document_type, source_document_id);

-- ============================================================================

-- 2. fin_acc_bank_reconciliations (مدیریت فرآیند مغایرت‌گیری بانکی پلتفرم)

-- ============================================================================

CREATE TABLE fin_acc_bank_reconciliations (

    reconciliation_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    bank_account_id UUID NOT NULL, -- ارجاع منطقی به حساب بانکی ماژول خزانه

    statement_date DATE NOT NULL,

    statement_balance NUMERIC(20,4) NOT NULL, -- مانده طبق صورتحساب بانک

    ledger_balance BIGINT NOT NULL DEFAULT 0, -- مانده طبق دفتر معین سیستم

    status SMALLINT NOT NULL DEFAULT 1, -- 1: Open, 2: Reconciled

    completed_at TIMESTAMPTZ,

    completed_by UUID,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    row_version BIGINT NOT NULL DEFAULT 1

);

-- ============================================================================

-- 3. fin_acc_budget_headers (سربرگ ساختار و کنترل بودجه‌های سازمانی)

-- ============================================================================

CREATE TABLE fin_acc_budget_headers (

    budget_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    fiscal_year_id UUID NOT NULL, -- ارجاع منطقی به سال مالی

    budget_code VARCHAR(50) NOT NULL,

    budget_name VARCHAR(150) NOT NULL,

    status SMALLINT NOT NULL DEFAULT 1, -- 1: Draft, 2: Approved, 3: Closed

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_fin_budgets_code ON fin_acc_budget_headers(tenant_id, budget_code) WHERE deleted_at IS NULL;

-- ============================================================================

-- 4. fin_acc_budget_items (ریز سطور و اقلام بودجه به تفکیک حساب و مرکز هزینه)

-- ============================================================================

CREATE TABLE fin_acc_budget_items (

    budget_item_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    budget_id UUID NOT NULL REFERENCES fin_acc_budget_headers(budget_id) ON DELETE RESTRICT,

    tenant_id UUID NOT NULL,

    account_id UUID NOT NULL, -- ارجاع فیزیکی به سرفصل حسابداری داخل همین ماژول

    cost_center_id UUID, -- ارجاع فیزیکی به مرکز هزینه داخل همین ماژول

    allocated_amount NUMERIC(20,4) NOT NULL,

    utilized_amount NUMERIC(20,4) NOT NULL DEFAULT 0.0000,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()

);

CREATE INDEX idx_fin_budget_items_lookup ON fin_acc_budget_items(budget_id);

-- ============================================================================

-- 5. fin_acc_fixed_assets (لجر اموال، دارایی‌های ثابت و تجهیزات سازمان)

-- ============================================================================

CREATE TABLE fin_acc_fixed_assets (

    asset_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    asset_code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    acquisition_date DATE NOT NULL,

    purchase_cost NUMERIC(20,4) NOT NULL,

    salvage_value NUMERIC(20,4) NOT NULL DEFAULT 0.0000, -- ارزش اسقاط

    depreciation_method SMALLINT NOT NULL DEFAULT 1, -- 1: Straight Line, 2: Declining Balance

    useful_life_months INT NOT NULL, -- عمر مفید به ماه

    accumulated_depreciation NUMERIC(20,4) NOT NULL DEFAULT 0.0000, -- استهلاک انباشته

    status SMALLINT NOT NULL DEFAULT 1, -- 1: Active, 2: Fully Depreciated, 3: Disposed

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_fin_assets_code ON fin_acc_fixed_assets(tenant_id, asset_code) WHERE deleted_at IS NULL;

-- ============================================================================

-- 6. fin_acc_depreciation_entries (اسناد و محاسبات دوره‌ای استهلاک دارایی‌ها)

-- ============================================================================

CREATE TABLE fin_acc_depreciation_entries (

    depreciation_entry_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    asset_id UUID NOT NULL REFERENCES fin_acc_fixed_assets(asset_id) ON DELETE RESTRICT,

    tenant_id UUID NOT NULL,

    journal_entry_id UUID, -- ارجاع منطقی به سند روزنامه صادر شده

    period_start DATE NOT NULL,

    period_end DATE NOT NULL,

    depreciation_amount NUMERIC(20,4) NOT NULL,

    calculated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()

);

-- ۱. ارتقای کدینگ حساب‌ها جهت کنترل لایه اپلیکیشن و ماهیت حسابداری

ALTER TABLE fin_acc_accounts 

ADD COLUMN normal_balance SMALLINT NOT NULL DEFAULT 1, -- 1: Debit (بدهکار), 2: Credit (بستانکار)

ADD COLUMN is_control_account BOOLEAN NOT NULL DEFAULT FALSE;

-- ۲. افزودن فیلدهای اتصال به موتور گردش کار پویا (تعریف‌شده در لایه ۵) جهت اتوماسیون تایید اسناد

ALTER TABLE fin_acc_journal_entries 

ADD COLUMN workflow_instance_id UUID, -- ارجاع منطقی به فرآیند زنده گردش کار

ADD COLUMN approval_status SMALLINT NOT NULL DEFAULT 1; -- 1: Draft, 2: Pending, 3: Approved

-- ۳. افزودن یک قید سخت‌گیرانه (CHECK Constraint) تا ثبت بدهکار و بستانکار هم‌زمان تداخل نداشته باشد

ALTER TABLE fin_acc_journal_items 

ADD CONSTRAINT chk_journal_item_amounts 

CHECK (

    (debit_amount > 0 AND credit_amount = 0) OR 

    (credit_amount > 0 AND debit_amount = 0)

);

-- ۴. افزایش اعشار فیلد نرخ برابری روزانه از ۴ رقم به ۸ رقم جهت پایش صرافی و تراکنش‌های بین‌المللی دقیق

ALTER TABLE fin_acc_exchange_rates 

ALTER COLUMN rate_value TYPE NUMERIC(20,8);

