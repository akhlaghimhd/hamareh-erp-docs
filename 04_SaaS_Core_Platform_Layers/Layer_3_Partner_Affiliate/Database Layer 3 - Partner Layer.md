# Database Layer 3 - Partner Layer

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** SaaS Core Platform Layers
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- =========================================================================

-- Layer 3: Partner Layer (REVISED & SECURED)

-- =========================================================================

-- ============================================================================

-- partners

-- SaaS Partner / Affiliate Management Layer

-- Separated from Layer 5 Business Partner Master Data

-- ============================================================================

CREATE TABLE partners (

    partner_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID,

    parent_partner_id UUID 

        REFERENCES partners(partner_id) ON DELETE RESTRICT,

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    partner_type SMALLINT NOT NULL DEFAULT 1,

    -- 1: Affiliate

    -- 2: Reseller

    -- 3: Implementation Partner

    -- 4: Service Partner

-- Replace uq_partners_tenant_code in Database Layer 3 - Partner Layer.sql

CREATE UNIQUE INDEX uq_partners_code_tenant 

    ON partners(code, COALESCE(tenant_id, '00000000-0000-0000-0000-000000000000'::uuid)) 

    WHERE deleted_at IS NULL;

    ownership_type SMALLINT NOT NULL DEFAULT 1,

    -- 1: Individual

    -- 2: Organization

    commission_enabled BOOLEAN NOT NULL DEFAULT TRUE,

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

CREATE UNIQUE INDEX uq_partners_tenant_code

ON partners(tenant_id, code)

WHERE deleted_at IS NULL;

CREATE INDEX idx_partners_parent

ON partners(parent_partner_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_partners_tenant

ON partners(tenant_id)

WHERE deleted_at IS NULL;

-- 2. partner_users

-- اصلاح معماری: فیلدهای فیزیکی دسترسی و نقش حذف شدند (ارجاع کامل به Layer 4)

-- Replace partner_users definition in Database Layer 3 - Partner Layer.sql

CREATE TABLE partner_users (

    partner_user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    partner_id UUID NOT NULL REFERENCES partners(partner_id) ON DELETE RESTRICT,

    user_id UUID NOT NULL, -- Standardized name directly (Logical Reference)

    is_primary BOOLEAN NOT NULL DEFAULT FALSE,

    status SMALLINT NOT NULL DEFAULT 1,

    

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_partner_users 

    ON partner_users(partner_id, user_id) 

    WHERE deleted_at IS NULL;-- 3. partner_tenant_assignments

CREATE TABLE partner_tenant_assignments (

 assignment_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 partner_id UUID NOT NULL REFERENCES partners(partner_id) ON DELETE RESTRICT,

 tenant_id UUID NOT NULL, -- ارجاع منطقی به لایه ساس ادمین بر اساس اصول مرزبندی

 assignment_type SMALLINT NOT NULL DEFAULT 1,

 start_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 end_date TIMESTAMPTZ,

 transfer_reason VARCHAR(500),

 assigned_by UUID,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_partner_assignments_tenant ON partner_tenant_assignments(tenant_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_partner_assignments_partner ON partner_tenant_assignments(partner_id) WHERE deleted_at IS NULL;

-- 4. partner_agreements

CREATE TABLE partner_agreements (

 agreement_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 partner_id UUID NOT NULL REFERENCES partners(partner_id) ON DELETE RESTRICT,

 agreement_number VARCHAR(100) NOT NULL,

 agreement_type SMALLINT NOT NULL DEFAULT 1,

 start_date TIMESTAMPTZ NOT NULL,

 end_date TIMESTAMPTZ,

 payment_cycle SMALLINT NOT NULL DEFAULT 1,

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

CREATE UNIQUE INDEX uq_partner_agreements_number ON partner_agreements(agreement_number) WHERE deleted_at IS NULL;

CREATE INDEX idx_partner_agreements_partner ON partner_agreements(partner_id) WHERE deleted_at IS NULL;

-- 5. partner_commission_rules

-- اصلاح محاسباتی: ارتقا به NUMERIC(20,4) برای مبالغ و ارزها جهت دقت بالا

CREATE TABLE partner_commission_rules (

 commission_rule_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 agreement_id UUID NOT NULL REFERENCES partner_agreements(agreement_id) ON DELETE RESTRICT, -- قفل لایه‌ای حذف

 revenue_type SMALLINT NOT NULL,

 commission_type SMALLINT NOT NULL,

 commission_value NUMERIC(20,4) NOT NULL,

 calculation_basis SMALLINT NOT NULL DEFAULT 1,

 minimum_amount NUMERIC(20,4),

 maximum_amount NUMERIC(20,4),

 effective_from TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 effective_to TIMESTAMPTZ,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_partner_comm_rules_agreement ON partner_commission_rules(agreement_id) WHERE deleted_at IS NULL;

-- 6. partner_commissions

CREATE TABLE partner_commissions (

 commission_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 partner_id UUID NOT NULL REFERENCES partners(partner_id) ON DELETE RESTRICT,

 tenant_id UUID NOT NULL, -- ارجاع منطقی لایه‌ای

 invoice_id UUID, -- رفرنس منطقی به فاکتور پلتفرم

 commission_rule_id UUID REFERENCES partner_commission_rules(commission_rule_id) ON DELETE RESTRICT,

 base_amount NUMERIC(20,4) NOT NULL,

 commission_type_snapshot SMALLINT NOT NULL,

 commission_value_snapshot NUMERIC(20,4) NOT NULL,

 commission_amount NUMERIC(20,4) NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 calculated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 paid_at TIMESTAMPTZ,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_partner_commissions_invoice ON partner_commissions(invoice_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_partner_commissions_partner ON partner_commissions(partner_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_partner_commissions_tenant ON partner_commissions(tenant_id) WHERE deleted_at IS NULL;

-- 7. partner_payouts

CREATE TABLE partner_payouts (

 payout_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 partner_id UUID NOT NULL REFERENCES partners(partner_id) ON DELETE RESTRICT,

 payout_number VARCHAR(100) NOT NULL,

 total_amount NUMERIC(20,4) NOT NULL,

 payout_date TIMESTAMPTZ,

 payment_reference VARCHAR(200),

 status SMALLINT NOT NULL DEFAULT 1,

 description VARCHAR(500),

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_partner_payouts_number ON partner_payouts(payout_number) WHERE deleted_at IS NULL;

CREATE INDEX idx_partner_payouts_partner ON partner_payouts(partner_id) WHERE deleted_at IS NULL;

-- ============================================================================

-- 1. partner_contacts (مدیریت مخاطبان و پرسنل کلیدی شرکای تجاری)

-- ============================================================================

CREATE TABLE partner_contacts (

    partner_contact_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    partner_id UUID NOT NULL, -- ارجاع به جدول اصلی پارتنرها

    first_name VARCHAR(100) NOT NULL,

    last_name VARCHAR(100) NOT NULL,

    role_title VARCHAR(100), -- سمت شغلی (مثلاً مدیر فنی، رابط مالی)

    email VARCHAR(255),

    phone_number VARCHAR(50),

    is_primary BOOLEAN NOT NULL DEFAULT FALSE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_partner_contacts_parent ON partner_contacts(partner_id);

-- ============================================================================

-- 2. partner_documents (مدارک، مجوزها و اسناد احراز هویت پارتنرها)

-- ============================================================================

CREATE TABLE partner_documents (

    partner_document_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    partner_id UUID NOT NULL,

    document_type VARCHAR(100) NOT NULL, -- NDA, Contract, Identification

    document_number VARCHAR(100),

    storage_path VARCHAR(1000) NOT NULL, -- آدرس فایل در مخزن امن پلتفرم

    status SMALLINT NOT NULL DEFAULT 1, -- 1: Pending Verification, 2: Approved, 3: Rejected

    verified_at TIMESTAMPTZ,

    verified_by UUID, -- ارجاع منطقی به ادمین تاییدکننده

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_partner_docs_parent ON partner_documents(partner_id);

-- ============================================================================

-- 3. partner_bank_accounts (حساب‌های بانکی پارتنرها جهت تسویه‌حساب)

-- ============================================================================

CREATE TABLE partner_bank_accounts (

    partner_bank_account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    partner_id UUID NOT NULL,

    bank_name VARCHAR(150) NOT NULL,

    account_number VARCHAR(50),

    shaba_number VARCHAR(50) NOT NULL, -- شماره شبا IR

    card_number VARCHAR(16),

    is_active BOOLEAN NOT NULL DEFAULT TRUE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_partner_bank_shaba ON partner_bank_accounts(shaba_number) WHERE is_active = TRUE;

-- ============================================================================

-- 4. partner_activity_logs (ردیابی و لاگ فعالیت‌های انجام‌شده در پورتال پارتنر)

-- ============================================================================

CREATE TABLE partner_activity_logs (

    partner_log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    partner_id UUID NOT NULL,

    user_id UUID NOT NULL, -- ارجاع منطقی به کاربر انجام‌دهنده در لایه هویت

    action_type VARCHAR(100) NOT NULL, -- e.g., 'WITHDRAWAL_REQUEST', 'PROFILE_UPDATE'

    description TEXT NOT NULL,

    ip_address VARCHAR(45) NOT NULL,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()

);

CREATE INDEX idx_partner_activity_parent ON partner_activity_logs(partner_id, created_at DESC);

-- ۱. اصلاح جدول پارتنرها برای افزودن فیلدهای ساختاری و ایزوله‌سازی کانتکست

ALTER TABLE partners 

ADD COLUMN tenant_id UUID, -- در صورت ارتباط با یک مستأجر خاص

ADD COLUMN parent_path TEXT, -- برای ساختارهای پارتنری درختی و چندسطحی (Multi-tier Marketing)

ADD COLUMN commission_enabled BOOLEAN NOT NULL DEFAULT TRUE;

-- ۲. اصلاح جدول کاربران پارتنر (تغییر نام فیلد جهت یکپارچگی با هویت هسته بر اساس تصویر)

ALTER TABLE partner_users 

RENAME COLUMN tenant_user_id TO user_id;

-- ۳. اصلاح جدول کارمزدهای پارتنر

ALTER TABLE partner_commissions 

ADD COLUMN currency_id UUID NOT NULL, -- ارجاع منطقی به جدول ارزها

ADD COLUMN exchange_rate NUMERIC(20,8) NOT NULL DEFAULT 1.00000000; -- پشتیبانی از اعشار دقیق طبق استاندارد

-- ۴. اصلاح جدول تسویه‌حساب‌ها و Payoutهای پارتنر

ALTER TABLE partner_payouts 

ADD COLUMN currency_id UUID NOT NULL,

ADD COLUMN bank_account_id UUID; -- ارجاع منطقی به حساب بانکی متصل پارتنر

