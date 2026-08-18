# 02 Master Data Table Definitions

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Identity & Master Data Layer
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- ============================================================================

-- Layer 5: ERP Core Master Data Layer (REVISED) - PART 1 OF 5

-- Main File: 02_Master_Data_Table_Definitions.docx

-- Database Engine: PostgreSQL 18 Enterprise Configuration

-- ============================================================================

CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- 1. business_partners (ریشه یکپارچه اطلاعات مشتریان، تامین‌کنندگان و پیمانکاران)

CREATE TABLE business_partners ( business_partner_id UUID PRIMARY KEY DEFAULT gen_random_uuid(), -- Multi Tenant Isolation tenant_id UUID NOT NULL, -- Identification code VARCHAR(50) NOT NULL, display_name VARCHAR(200) NOT NULL, -- Partner Classification partner_type SMALLINT NOT NULL DEFAULT 1, -- 1: Individual -- 2: Organization status SMALLINT NOT NULL DEFAULT 1, -- 1: Active -- 2: Suspended -- 3: Blocked -- Optional Parent Relationship parent_business_partner_id UUID REFERENCES business_partners(business_partner_id) ON DELETE RESTRICT, -- Audit Fields created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(), created_by UUID, updated_at TIMESTAMPTZ, updated_by UUID, deleted_at TIMESTAMPTZ, deleted_by UUID, row_version BIGINT NOT NULL DEFAULT 1 ); -- Unique Business Partner Code per Tenant CREATE UNIQUE INDEX uq_business_partners_tenant_code ON business_partners(tenant_id, code) WHERE deleted_at IS NULL; -- Tenant Query Optimization CREATE INDEX idx_business_partners_tenant ON business_partners(tenant_id) WHERE deleted_at IS NULL; -- Search Optimization CREATE INDEX idx_business_partners_display_name ON business_partners(display_name) WHERE deleted_at IS NULL; -- Parent Hierarchy Support CREATE INDEX idx_business_partners_parent ON business_partners(parent_business_partner_id) WHERE deleted_at IS NULL;

-- 2. persons (پروفایل اطلاعات اشخاص حقیقی متصل به شریک تجاری)

CREATE TABLE persons (

 person_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 business_partner_id UUID NOT NULL REFERENCES business_partners(business_partner_id) ON DELETE RESTRICT, -- قفل سخت‌گیرانه برای حفظ دیتای مالی

 first_name VARCHAR(100) NOT NULL,

 last_name VARCHAR(100) NOT NULL,

 national_code VARCHAR(50),

 birth_date DATE,

 gender SMALLINT,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_persons_business_partner ON persons(business_partner_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_persons_national_code ON persons(national_code) WHERE deleted_at IS NULL;

-- 3. business_partner_organizations (پروفایل اطلاعات شرکت‌ها و سازمان‌های حقوقی)

CREATE TABLE business_partner_organizations (

 business_partner_organization_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 business_partner_id UUID NOT NULL REFERENCES business_partners(business_partner_id) ON DELETE RESTRICT,

 legal_name VARCHAR(200) NOT NULL,

 trade_name VARCHAR(200),

 registration_number VARCHAR(100),

 organization_type SMALLINT,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_bp_organizations_business_partner ON business_partner_organizations(business_partner_id) WHERE deleted_at IS NULL;

-- ============================================================================

-- 4. business_partner_roles

-- Business Partner Role Pattern

-- Supports Customer, Supplier, Contractor, Carrier, Service Provider, etc.

-- ============================================================================

CREATE TABLE business_partner_roles (

    business_partner_role_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    business_partner_id UUID NOT NULL 

        REFERENCES business_partners(business_partner_id) 

        ON DELETE RESTRICT,

    role_type SMALLINT NOT NULL,

    /*

        1  Customer

        2  Supplier

        3  Contractor

        4  Carrier

        5  Service Provider

        6  Manufacturer

        7  Distributor

        8  Agent

        9  Employee

        10 Other

    */

    role_code VARCHAR(50) NOT NULL,

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

CREATE UNIQUE INDEX uq_business_partner_roles_active

ON business_partner_roles(

    business_partner_id,

    role_type

)

WHERE deleted_at IS NULL;

CREATE UNIQUE INDEX uq_business_partner_role_code

ON business_partner_roles(

    business_partner_id,

    role_code

)

WHERE deleted_at IS NULL;

CREATE INDEX idx_business_partner_roles_partner

ON business_partner_roles(

    business_partner_id

)

WHERE deleted_at IS NULL;

CREATE INDEX idx_business_partner_roles_type

ON business_partner_roles(

    role_type

)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 5. business_partner_contacts

-- مدیریت مخاطبین مستقل Business Partner

-- ============================================================================

CREATE TABLE business_partner_contacts (

    business_partner_contact_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    business_partner_id UUID NOT NULL

        REFERENCES business_partners(business_partner_id)

        ON DELETE RESTRICT,

    contact_type SMALLINT NOT NULL,

    /*

        1 Primary Contact

        2 Financial Contact

        3 Technical Contact

        4 Sales Contact

        5 Support Contact

        6 Legal Contact

    */

    first_name VARCHAR(100) NOT NULL,

    last_name VARCHAR(100) NOT NULL,

    job_title VARCHAR(150),

    email VARCHAR(255),

    mobile VARCHAR(50),

    phone VARCHAR(50),

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

CREATE INDEX idx_bp_contacts_partner

ON business_partner_contacts(business_partner_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_bp_contacts_email

ON business_partner_contacts(email)

WHERE deleted_at IS NULL;

CREATE UNIQUE INDEX uq_bp_primary_contact

ON business_partner_contacts(

    business_partner_id,

    contact_type

)

WHERE is_primary = TRUE

AND deleted_at IS NULL;

-- 5. business_partner_identifications (مدیریت مدارک شناسایی مانند شناسه اقتصادی یا گذرنامه)

CREATE TABLE business_partner_identifications (

 bp_identification_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 business_partner_id UUID NOT NULL REFERENCES business_partners(business_partner_id) ON DELETE RESTRICT,

 tenant_id UUID NOT NULL,

 identification_type_code VARCHAR(50) NOT NULL, -- (e.g., 'ECONOMIC_CODE', 'PASSPORT')

 id_number VARCHAR(100) NOT NULL,

 issue_date DATE,

 expiry_date DATE,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 deleted_at TIMESTAMPTZ

);

CREATE UNIQUE INDEX uq_bp_identifications_tenant ON business_partner_identifications(tenant_id, identification_type_code, id_number) WHERE deleted_at IS NULL;

CREATE INDEX idx_bp_identifications_partner ON business_partner_identifications(business_partner_id) WHERE deleted_at IS NULL;

-- ============================================================================

-- Layer 5: ERP Core Master Data Layer (REVISED) - PART 2 OF 5

-- Main File: 02_Master_Data_Table_Definitions.docx

-- Database Engine: PostgreSQL 18

-- ============================================================================

-- 6. wf_process_definitions (تعریف شابلون فرآیندها - خط تولید یا زنجیره خدمات)

CREATE TABLE wf_process_definitions (

 process_definition_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL, -- قفل در بافتار چندمستأجری امن

 code VARCHAR(100) NOT NULL, -- (e.g., 'MANUFACTURING_FLOW_V1', 'CYBER_SERVICE_FLOW')

 name VARCHAR(200) NOT NULL,

 target_aggregate_type VARCHAR(100) NOT NULL, -- (موجودیتی که فرآیند رویش اجرا میشود)

 flow_graph JSONB NOT NULL, -- ذخیره کل مراحل، شرط‌ها و مسیرهای بازگشت به صورت گراف پویا

 is_active BOOLEAN NOT NULL DEFAULT TRUE,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_wf_process_def ON wf_process_definitions(tenant_id, code);

-- 7. wf_process_instances (موتور ردیابی زنده وضعیت هر سند در گردش کار)

CREATE TABLE wf_process_instances (

 process_instance_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 process_definition_id UUID NOT NULL REFERENCES wf_process_definitions(process_definition_id) ON DELETE RESTRICT,

 target_aggregate_id UUID NOT NULL, -- شناسه فیزیکی فاکتور، سند حسابداری یا سفارش تولید (Logical Reference)

 current_state VARCHAR(100) NOT NULL, -- وضعیت فعلی در گراف فرآیند

 owning_tenant_id UUID NOT NULL, -- شرکتی که این مرحله را مالک است (حیاتی برای اتوماسیون بین‌شرکتی)

 status SMALLINT NOT NULL DEFAULT 1, -- (1: Running, 2: Completed, 3: Terminated/Failed)

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 updated_at TIMESTAMPTZ

);

CREATE INDEX idx_wf_instances_target ON wf_process_instances(target_aggregate_id);

CREATE INDEX idx_wf_instances_owner ON wf_process_instances(owning_tenant_id, current_state);

-- 8. wf_tasks (کارپوشه پورتال - ارجاع تسک‌ها به کارمندان داخلی یا پیمانکاران خارجی بخش اول)

CREATE TABLE wf_tasks (

 task_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 process_instance_id UUID NOT NULL REFERENCES wf_process_instances(process_instance_id) ON DELETE RESTRICT,

 assigned_type SMALLINT NOT NULL, -- (1: Internal Role, 2: External Business Partner / Contractor)

 assigned_to_id UUID NOT NULL, -- اگر ۱ باشد شناسه role_id لایه ۴ و اگر ۲ باشد business_partner_id بخش اول است\!

 task_name VARCHAR(200) NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1, -- (1: Pending, 2: Approved, 3: Rejected)

 context_snapshots JSONB, -- ذخیره اطلاعات خلاصه سند برای نمایش سریع در فرانت‌اند بدون JOIN سنگین

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 actioned_at TIMESTAMPTZ,

 actioned_by UUID 

);

CREATE INDEX idx_wf_tasks_assignment ON wf_tasks(tenant_id, assigned_type, assigned_to_id) WHERE status = 1;

-- 9. entity_addresses (جدول آدرس‌های چندشکلی چندمستأجری برای شرکا، شعب یا شرکت‌ها)

CREATE TABLE entity_addresses (

 entity_address_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 entity_type VARCHAR(100) NOT NULL, -- (e.g., 'BUSINESS_PARTNER', 'BRANCH', 'COMPANY')

 entity_id UUID NOT NULL, -- شناسه پویای لایه‌ای

 address_type_id UUID NOT NULL, -- متصل به مقادیر پایه بخش چهارم

 country_id UUID NOT NULL, -- ارجاع منطقی گلوبال به بخش چهارم

 province_name VARCHAR(150),

 city_name VARCHAR(150),

 postal_code VARCHAR(50),

 address_text TEXT NOT NULL,

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

CREATE INDEX idx_entity_addresses_polymorphic ON entity_addresses(entity_id, entity_type) WHERE deleted_at IS NULL;

CREATE INDEX idx_entity_addresses_tenant ON entity_addresses(tenant_id) WHERE deleted_at IS NULL;

-- 10. entity_contact_points (جدول چندشکلی اطلاعات تماس - تلفن، ایمیل، وب‌سایت)

CREATE TABLE entity_contact_points (

 entity_contact_point_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 entity_type VARCHAR(100) NOT NULL,

 entity_id UUID NOT NULL,

 contact_point_type SMALLINT NOT NULL, -- (1: Phone, 2: Mobile, 3: Email, 4: Website)

 contact_value VARCHAR(255) NOT NULL,

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

CREATE INDEX idx_entity_contact_polymorphic ON entity_contact_points(entity_id, entity_type) WHERE deleted_at IS NULL;

-- ============================================================================

-- Layer 5: ERP Core Master Data Layer (REVISED) - PART 3 OF 5

-- Main File: 02_Master_Data_Table_Definitions.docx

-- Database Engine: PostgreSQL 18

-- ============================================================================

-- 11. fiscal_years (مدیریت سال‌های مالی هر شرکت به صورت ایزوله چندمستأجری)

CREATE TABLE fiscal_years (

 fiscal_year_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL, -- متصل به بافتار مالتی-تننت امن پلتفرم

 company_id UUID NOT NULL, -- ارجاع منطقی به لایه ۴ (بدون FK فیزیکی بر اساس اصول مرزبندی)

 year_code VARCHAR(50) NOT NULL, -- (e.g., "1405", "2026")

 name VARCHAR(200) NOT NULL,

 start_date DATE NOT NULL,

 end_date DATE NOT NULL,

 is_closed BOOLEAN NOT NULL DEFAULT FALSE,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_fiscal_years_tenant_code ON fiscal_years(tenant_id, company_id, year_code) WHERE deleted_at IS NULL;

CREATE INDEX idx_fiscal_years_company ON fiscal_years(company_id) WHERE deleted_at IS NULL;

-- 12. fiscal_periods (ماه یا دوره‌های مالی عملیاتی درون سال مالی)

CREATE TABLE fiscal_periods (

 fiscal_period_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 fiscal_year_id UUID NOT NULL REFERENCES fiscal_years(fiscal_year_id) ON DELETE RESTRICT, -- قفل از حذف آبشاری زنجیره‌ای مالی

 period_number SMALLINT NOT NULL, -- (1-12)

 name VARCHAR(100) NOT NULL,

 start_date DATE NOT NULL,

 end_date DATE NOT NULL,

 is_closed BOOLEAN NOT NULL DEFAULT FALSE,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_fiscal_periods_sequence ON fiscal_periods(fiscal_year_id, period_number) WHERE deleted_at IS NULL;

-- 13. shipment_methods (روش‌های حمل و تحویل کالا کاتالوگ لوکاپ)

CREATE TABLE shipment_methods (

 shipment_method_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 code VARCHAR(50) NOT NULL, -- (e.g., "AIR", "SEA", "EXPRESS_POST")

 name VARCHAR(150) NOT NULL,

 description VARCHAR(500),

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 deleted_at TIMESTAMPTZ

);

CREATE UNIQUE INDEX uq_shipment_methods_tenant_code ON shipment_methods(tenant_id, code) WHERE deleted_at IS NULL;

-- 14. payment_terms (ترم‌ها و شرایط پرداخت فاکتورها - سقف اعتباری و نقدی)

-- اصلاح محاسباتی: ارتقا به NUMERIC(20,4) برای مبالغ و نرخ جریمه‌ها جهت تضمین صحت محاسبات

CREATE TABLE payment_terms (

 payment_term_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 code VARCHAR(50) NOT NULL, -- (e.g., "NET_30", "2_10_NET_30")

 name VARCHAR(200) NOT NULL,

 description VARCHAR(500),

 net_days INT NOT NULL DEFAULT 0,

 discount_days INT NOT NULL DEFAULT 0,

 discount_percentage NUMERIC(20,4) NOT NULL DEFAULT 0,

 penalty_percentage_per_month NUMERIC(20,4) NOT NULL DEFAULT 0,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_payment_terms_tenant_code ON payment_terms(tenant_id, code) WHERE deleted_at IS NULL;

-- ============================================================================

-- Layer 5: ERP Core Master Data Layer (REVISED) - PART 4 OF 5

-- Main File: 02_Master_Data_Table_Definitions.docx

-- Database Engine: PostgreSQL 18

-- ============================================================================

-- 15. units_of_measure (واحدهای سنجش انبار و کالا - UOM)

-- اصلاح امنیت ساس: ستون tenant_id اضافه شد و ایندکس ترکیبی شد تا تداخل گلوبال رخ ندهد.

CREATE TABLE units_of_measure (

 uom_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL, 

 code VARCHAR(50) NOT NULL, -- (e.g., "KG", "BOX", "METER")

 name VARCHAR(100) NOT NULL,

 decimal_places SMALLINT NOT NULL DEFAULT 0,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_uom_tenant_code ON units_of_measure(tenant_id, code) WHERE deleted_at IS NULL;

CREATE INDEX idx_uom_tenant ON units_of_measure(tenant_id) WHERE deleted_at IS NULL;

-- 16. countries (کاتالوگ ثابت و عمومی پلتفرم - بدون نیاز به tenant_id)

CREATE TABLE countries (

 country_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 iso_code VARCHAR(10) NOT NULL, -- (e.g., "IR", "DE", "US")

 name VARCHAR(150) NOT NULL,

 phone_code VARCHAR(20),

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 deleted_at TIMESTAMPTZ

);

CREATE UNIQUE INDEX uq_countries_iso ON countries(iso_code) WHERE deleted_at IS NULL;

-- ============================================================================

-- 20. currencies

-- Currency Master Data

-- ============================================================================

CREATE TABLE currencies (

    currency_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    code VARCHAR(10) NOT NULL,

    -- مثال: USD, EUR, IRR

    name VARCHAR(100) NOT NULL,

    symbol VARCHAR(10),

    decimal_places SMALLINT NOT NULL DEFAULT 2,

    is_base_currency BOOLEAN NOT NULL DEFAULT FALSE,

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_currencies_code

ON currencies(code)

WHERE deleted_at IS NULL;

CREATE INDEX idx_currencies_status

ON currencies(status)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 21. exchange_rates

-- Multi Currency Exchange Rate Master Data

-- ============================================================================

CREATE TABLE exchange_rates (

    exchange_rate_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    from_currency_id UUID NOT NULL

        REFERENCES currencies(currency_id)

        ON DELETE RESTRICT,

    to_currency_id UUID NOT NULL

        REFERENCES currencies(currency_id)

        ON DELETE RESTRICT,

    rate NUMERIC(20,8) NOT NULL,

    effective_from TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    effective_to TIMESTAMPTZ,

    rate_type SMALLINT NOT NULL DEFAULT 1,

    /*

        1 Official

        2 Market

        3 Custom

    */

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_exchange_rates_from_currency

ON exchange_rates(from_currency_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_exchange_rates_to_currency

ON exchange_rates(to_currency_id)

WHERE deleted_at IS NULL;

CREATE UNIQUE INDEX uq_exchange_rate_period

ON exchange_rates(

    from_currency_id,

    to_currency_id,

    effective_from

)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 22. chart_of_accounts

-- Financial Account Master Data

-- ============================================================================

CREATE TABLE chart_of_accounts (

    account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    company_id UUID NOT NULL,

    parent_account_id UUID

        REFERENCES chart_of_accounts(account_id)

        ON DELETE RESTRICT,

    account_code VARCHAR(50) NOT NULL,

    account_name VARCHAR(200) NOT NULL,

    account_type SMALLINT NOT NULL,

    /*

        1 Asset

        2 Liability

        3 Equity

        4 Revenue

        5 Expense

    */

    account_level SMALLINT NOT NULL DEFAULT 1,

    is_postable BOOLEAN NOT NULL DEFAULT TRUE,

    -- آیا سند مالی مستقیم روی این حساب ثبت می‌شود

    normal_balance SMALLINT NOT NULL DEFAULT 1,

    /*

        1 Debit

        2 Credit

    */

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_chart_accounts_code

ON chart_of_accounts(

    tenant_id,

    company_id,

    account_code

)

WHERE deleted_at IS NULL;

CREATE INDEX idx_chart_accounts_parent

ON chart_of_accounts(parent_account_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_chart_accounts_company

ON chart_of_accounts(company_id)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 23. banks

-- Financial Master Data - Bank Directory

-- ============================================================================

CREATE TABLE banks (

    bank_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    country_id UUID,

    swift_code VARCHAR(50),

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_banks_code

ON banks(code)

WHERE deleted_at IS NULL;

CREATE INDEX idx_banks_country

ON banks(country_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_banks_status

ON banks(status)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 24. items

-- Inventory Master Data - Core Item Definition

-- ============================================================================

CREATE TABLE items (

    item_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    item_category_id UUID,

    uom_id UUID NOT NULL,

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    item_type SMALLINT NOT NULL,

    /*

        1 Stock Item

        2 Service

        3 Non Stock Item

        4 Consumable

        5 Asset

    */

    description TEXT,

    is_inventory_item BOOLEAN NOT NULL DEFAULT TRUE,

    is_purchase_item BOOLEAN NOT NULL DEFAULT TRUE,

    is_sales_item BOOLEAN NOT NULL DEFAULT FALSE,

    track_serial_number BOOLEAN NOT NULL DEFAULT FALSE,

    track_batch_number BOOLEAN NOT NULL DEFAULT FALSE,

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_items_tenant_code

ON items(

    tenant_id,

    code

)

WHERE deleted_at IS NULL;

CREATE INDEX idx_items_category

ON items(item_category_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_items_uom

ON items(uom_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_items_status

ON items(status)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 25. item_categories

-- Inventory Master Data - Item Classification

-- ============================================================================

CREATE TABLE item_categories (

    item_category_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    parent_category_id UUID

        REFERENCES item_categories(item_category_id)

        ON DELETE RESTRICT,

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    description VARCHAR(500),

    category_type SMALLINT NOT NULL DEFAULT 1,

    /*

        1 Stock

        2 Service

        3 Asset

        4 Consumable

    */

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_item_categories_tenant_code

ON item_categories(

    tenant_id,

    code

)

WHERE deleted_at IS NULL;

CREATE INDEX idx_item_categories_parent

ON item_categories(parent_category_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_item_categories_tenant

ON item_categories(tenant_id)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 26. products

-- Inventory Master Data - Sales Product Definition

-- ============================================================================

CREATE TABLE products (

    product_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    item_id UUID NOT NULL

        REFERENCES items(item_id)

        ON DELETE RESTRICT,

    product_code VARCHAR(50) NOT NULL,

    product_name VARCHAR(200) NOT NULL,

    product_type SMALLINT NOT NULL DEFAULT 1,

    /*

        1 Finished Product

        2 Service Product

        3 Subscription Product

        4 Bundle Product

    */

    sales_description TEXT,

    purchase_description TEXT,

    default_sales_price NUMERIC(20,4),

    default_purchase_price NUMERIC(20,4),

    tax_category_id UUID,

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_products_tenant_code

ON products(

    tenant_id,

    product_code

)

WHERE deleted_at IS NULL;

CREATE INDEX idx_products_item

ON products(item_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_products_tax_category

ON products(tax_category_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_products_status

ON products(status)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 27. warehouses

-- Inventory Master Data - Warehouse Definition

-- ============================================================================

CREATE TABLE warehouses (

    warehouse_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    company_id UUID NOT NULL,

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    warehouse_type SMALLINT NOT NULL DEFAULT 1,

    /*

        1 Main Warehouse

        2 Branch Warehouse

        3 Transit Warehouse

        4 Virtual Warehouse

    */

    address_id UUID,

    manager_user_id UUID,

    allow_negative_stock BOOLEAN NOT NULL DEFAULT FALSE,

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_warehouses_tenant_code

ON warehouses(

    tenant_id,

    code

)

WHERE deleted_at IS NULL;

CREATE INDEX idx_warehouses_company

ON warehouses(company_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_warehouses_status

ON warehouses(status)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 28. warehouse_locations

-- Inventory Master Data - Bin / Storage Location Definition

-- ============================================================================

CREATE TABLE warehouse_locations (

    warehouse_location_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    warehouse_id UUID NOT NULL

        REFERENCES warehouses(warehouse_id)

        ON DELETE RESTRICT,

    parent_location_id UUID

        REFERENCES warehouse_locations(warehouse_location_id)

        ON DELETE RESTRICT,

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    location_type SMALLINT NOT NULL DEFAULT 1,

    /*

        1 Zone

        2 Aisle

        3 Rack

        4 Shelf

        5 Bin

    */

    is_storage_location BOOLEAN NOT NULL DEFAULT TRUE,

    capacity_quantity NUMERIC(20,4),

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_warehouse_locations_code

ON warehouse_locations(

    warehouse_id,

    code

)

WHERE deleted_at IS NULL;

CREATE INDEX idx_warehouse_locations_parent

ON warehouse_locations(parent_location_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_warehouse_locations_warehouse

ON warehouse_locations(warehouse_id)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 29. tax_definitions

-- Financial Master Data - Tax Rules Definition

-- ============================================================================

CREATE TABLE tax_definitions (

    tax_definition_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    tax_category_id UUID

        REFERENCES tax_categories(tax_category_id)

        ON DELETE RESTRICT,

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    tax_type SMALLINT NOT NULL,

    /*

        1 VAT

        2 Sales Tax

        3 Purchase Tax

        4 Withholding Tax

        5 Other

    */

    calculation_type SMALLINT NOT NULL DEFAULT 1,

    /*

        1 Percentage

        2 Fixed Amount

    */

    tax_rate NUMERIC(20,4) NOT NULL DEFAULT 0,

    fixed_amount NUMERIC(20,4),

    effective_from TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    effective_to TIMESTAMPTZ,

    is_inclusive BOOLEAN NOT NULL DEFAULT FALSE,

    -- آیا مالیات داخل مبلغ نهایی محاسبه شده است

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_tax_definitions_tenant_code

ON tax_definitions(

    tenant_id,

    code

)

WHERE deleted_at IS NULL;

CREATE INDEX idx_tax_definitions_category

ON tax_definitions(tax_category_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_tax_definitions_status

ON tax_definitions(status)

WHERE deleted_at IS NULL;

-- 17. address_types (انواع متادیتای آدرس - متصل به جدول چندشکلی آدرس‌ها)

CREATE TABLE address_types (

 address_type_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 code VARCHAR(50) NOT NULL, -- (e.g., "OFFICE", "FACTORY", "WAREHOUSE_DELIVERY")

 name VARCHAR(100) NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 deleted_at TIMESTAMPTZ

);

CREATE UNIQUE INDEX uq_address_types_tenant_code ON address_types(tenant_id, code) WHERE deleted_at IS NULL;

CREATE INDEX idx_address_types_tenant ON address_types(tenant_id) WHERE deleted_at IS NULL;

-- 18. bank_accounts (مدیریت اطلاعات و شماره حساب‌های بانکی شرکت‌ها یا شرکای تجاری)

CREATE TABLE bank_accounts (

 bank_account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 entity_type VARCHAR(100) NOT NULL, -- (e.g., 'COMPANY', 'BUSINESS_PARTNER')

 entity_id UUID NOT NULL, -- Logical Reference پویا

 bank_name VARCHAR(150) NOT NULL,

 branch_name VARCHAR(150),

 account_number VARCHAR(100) NOT NULL,

 card_number VARCHAR(50),

 iban VARCHAR(100), -- شماره شبا

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

CREATE INDEX idx_bank_accounts_polymorphic ON bank_accounts(entity_id, entity_type) WHERE deleted_at IS NULL;

CREATE INDEX idx_bank_accounts_tenant ON bank_accounts(tenant_id) WHERE deleted_at IS NULL;

-- 19. tax_categories (دسته‌بندی‌های کلان قوانین مالیاتی و عوارض کالا)

CREATE TABLE tax_categories (

 tax_category_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL,

 code VARCHAR(50) NOT NULL, -- (e.g., "VAT_9", "VAT_10", "EXEMPT")

 name VARCHAR(150) NOT NULL,

 description VARCHAR(500),

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 deleted_at TIMESTAMPTZ

);

CREATE UNIQUE INDEX uq_tax_categories_tenant_code ON tax_categories(tenant_id, code) WHERE deleted_at IS NULL;

-- ============================================================================

-- document_sequences

-- ERP Document Numbering Engine

-- Centralized Dynamic Sequence Management

-- ============================================================================

CREATE TABLE document_sequences (

    sequence_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    company_id UUID,

    module_code VARCHAR(50) NOT NULL,

    /*

        FIN

        INV

        PUR

        SAL

        HR

        MFG

    */

    document_type VARCHAR(100) NOT NULL,

    /*

        SALES_INVOICE

        PURCHASE_ORDER

        STOCK_RECEIPT

        JOURNAL_ENTRY

    */

    document_scope SMALLINT NOT NULL DEFAULT 1,

    /*

        1 Tenant

        2 Company

        3 Branch

        4 Module

    */

    owner_type VARCHAR(50),

    /*

        COMPANY

        BRANCH

        MODULE

    */

    owner_id UUID,

    prefix VARCHAR(20),

    suffix VARCHAR(20),

    padding_length INT NOT NULL DEFAULT 6,

    current_value BIGINT NOT NULL DEFAULT 0,

    reset_period SMALLINT NOT NULL DEFAULT 1,

    /*

        1 Never

        2 Yearly

        3 Monthly

    */

    last_reset_at TIMESTAMPTZ,

    is_active BOOLEAN NOT NULL DEFAULT TRUE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_document_sequences_scope

ON document_sequences(

    tenant_id,

    module_code,

    document_type,

    document_scope,

    COALESCE(company_id, '00000000-0000-0000-0000-000000000000'::UUID)

)

WHERE is_active = TRUE;

CREATE INDEX idx_document_sequences_module

ON document_sequences(

    tenant_id,

    module_code

)

WHERE is_active = TRUE;

CREATE INDEX idx_document_sequences_owner

ON document_sequences(owner_id)

WHERE is_active = TRUE;

 -- ============================================================================

-- attachments

-- Core File Management Service

-- Shared Attachment Metadata Across ERP Modules

-- ============================================================================

CREATE TABLE attachments (

    attachment_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    entity_type VARCHAR(100) NOT NULL,

    -- نام موجودیت مقصد

    -- مثال: INVOICE, PURCHASE_ORDER, EMPLOYEE

    entity_id UUID NOT NULL,

    -- شناسه رکورد مقصد

    file_name VARCHAR(255) NOT NULL,

    original_file_name VARCHAR(255),

    file_extension VARCHAR(20),

    mime_type VARCHAR(100) NOT NULL,

    file_size_bytes BIGINT NOT NULL,

    storage_provider VARCHAR(50) NOT NULL DEFAULT 'LOCAL',

    /*

        LOCAL

        S3

        AZURE

        MINIO

    */

    storage_path VARCHAR(1000) NOT NULL,

    checksum VARCHAR(255),

    description VARCHAR(500),

    uploaded_by UUID,

    uploaded_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    status SMALLINT NOT NULL DEFAULT 1,

    /*

        1 Active

        2 Archived

        3 Deleted

    */

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_attachments_entity

ON attachments(

    tenant_id,

    entity_type,

    entity_id

)

WHERE deleted_at IS NULL;

CREATE INDEX idx_attachments_checksum

ON attachments(checksum)

WHERE deleted_at IS NULL;

CREATE INDEX idx_attachments_status

ON attachments(status)

WHERE deleted_at IS NULL;

-- ============================================================================

-- tags

-- ERP Dynamic Tagging Master Data

-- Tenant Scoped Tag Management

-- ============================================================================

CREATE TABLE tags (

    tag_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    scope_type VARCHAR(50) NOT NULL,

    /*

        SYSTEM

        MODULE

        ENTITY

    */

    module_code VARCHAR(50),

    /*

        INV

        PUR

        SAL

        FIN

        HR

    */

    entity_type VARCHAR(100),

    /*

        محدودسازی Tag برای موجودیت خاص

        مثال:

        INVOICE

        ITEM

        CUSTOMER

    */

    tag_name VARCHAR(100) NOT NULL,

    description VARCHAR(500),

    color_code VARCHAR(20),

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_tags_scope_name

ON tags(

    tenant_id,

    scope_type,

    COALESCE(module_code, ''),

    COALESCE(entity_type, ''),

    tag_name

)

WHERE deleted_at IS NULL;

CREATE INDEX idx_tags_tenant

ON tags(tenant_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_tags_module

ON tags(module_code)

WHERE deleted_at IS NULL;

CREATE INDEX idx_tags_entity_type

ON tags(entity_type)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 4. entity_tags (جدول واسط چندشکلی نگاشت برچسب‌ها به رکوردهای مختلف سیستم)

-- ============================================================================

CREATE TABLE entity_tags (

    entity_tag_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    tag_id UUID NOT NULL REFERENCES tags(tag_id) ON DELETE RESTRICT,

    target_entity_type VARCHAR(100) NOT NULL, -- نام جدول هدف (مثلا 'pur_orders')

    target_entity_id UUID NOT NULL, -- شناسه ردیف هدف

    assigned_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    assigned_by UUID

);

CREATE UNIQUE INDEX uq_entity_tags_mapping ON entity_tags(tenant_id, tag_id, target_entity_type, target_entity_id);

CREATE INDEX idx_entity_tags_lookup ON entity_tags(tenant_id, target_entity_type, target_entity_id);

-- ============================================================================

-- 5. employees (جدول اطلاعات و شناسنامه پرسنل سازمان)

-- ============================================================================

CREATE TABLE employees (

    employee_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    user_id UUID, -- ارجاع منطقی اختیاری به جدول کاربران سیستم هویت

    employee_code VARCHAR(50) NOT NULL,

    first_name VARCHAR(100) NOT NULL,

    last_name VARCHAR(100) NOT NULL,

    national_code VARCHAR(10) NOT NULL,

    hire_date DATE NOT NULL,

    termination_date DATE,

    status SMALLINT NOT NULL DEFAULT 1, -- 1: Active, 2: Suspended, 3: Terminated

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_employees_code ON employees(tenant_id, employee_code) WHERE deleted_at IS NULL;

CREATE UNIQUE INDEX uq_employees_national ON employees(tenant_id, national_code) WHERE deleted_at IS NULL;

-- ۱. پایش و سخت‌گیرانه کردن قوانین ایزوله‌سازی چندمستأجری در شرکای تجاری

ALTER TABLE business_partners 

ALTER COLUMN tenant_id SET NOT NULL;

-- ۲. افزودن ستون‌های حیاتی اطلاعات تماس به جدول اشخاص حقیقی

ALTER TABLE persons 

ADD COLUMN mobile VARCHAR(20),

ADD COLUMN email VARCHAR(255);

-- ۳. افزودن شناسه ملی/مالیاتی به اشخاص حقوقی

ALTER TABLE business_partner_organizations 

ADD COLUMN tax_identifier VARCHAR(50);

-- ۴. اصلاح جدول آرس‌ها (جایگزینی فیلدهای متنی استان/شهر با شناسه‌های ساختاریافته)

ALTER TABLE entity_addresses 

DROP COLUMN IF EXISTS province_name,

DROP COLUMN IF EXISTS city_name,

ADD COLUMN province_id UUID, -- ارجاع منطقی به کاتالوگ تقسیمات کشوری

ADD COLUMN city_id UUID;

-- ۵. افزودن فلگ تایید هویت برای کانال‌های ارتباطی

ALTER TABLE entity_contact_points 

ADD COLUMN verified BOOLEAN NOT NULL DEFAULT FALSE;

-- ۶. اصلاح جداول کاتالوگ و دوره‌های عملیاتی سیستم

ALTER TABLE fiscal_years ADD COLUMN currency_id UUID;

ALTER TABLE payment_terms ADD COLUMN installment_count INT NOT NULL DEFAULT 1;

ALTER TABLE units_of_measure ADD COLUMN conversion_factor NUMERIC(20,4) NOT NULL DEFAULT 1.0000;

ALTER TABLE countries ADD COLUMN iso_numeric_code VARCHAR(3);

-- 6. companies (Master Data سازمان‌ها و شرکت‌های زیرمجموعه Tenant)

-- ============================================================================

-- Layer 5: ERP Core Master Data Layer

-- Organization Master Data

-- 4. companies

-- ============================================================================

CREATE TABLE companies (

    company_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    legal_name VARCHAR(300),

    registration_number VARCHAR(100),

    tax_identifier VARCHAR(100),

    company_type SMALLINT NOT NULL DEFAULT 1,

    -- 1: Legal Entity

    -- 2: Business Unit

    currency_id UUID,

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_companies_tenant_code

ON companies(tenant_id, code)

WHERE deleted_at IS NULL;

CREATE INDEX idx_companies_tenant

ON companies(tenant_id)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 7. branches (Master Data شعب شرکت‌ها)

-- ============================================================================

CREATE TABLE branches (

    branch_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    company_id UUID NOT NULL,

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    branch_type SMALLINT NOT NULL DEFAULT 1,

    manager_user_id UUID,

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_branches_company_code

ON branches(company_id, code)

WHERE deleted_at IS NULL;

CREATE INDEX idx_branches_tenant

ON branches(tenant_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_branches_company

ON branches(company_id)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 8. departments (Master Data واحدها و دپارتمان‌های سازمان)

-- ============================================================================

CREATE TABLE departments (

    department_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    company_id UUID NOT NULL,

    branch_id UUID,

    parent_department_id UUID,

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    manager_employee_id UUID,

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_departments_company_code

ON departments(company_id, code)

WHERE deleted_at IS NULL;

CREATE INDEX idx_departments_tenant

ON departments(tenant_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_departments_company

ON departments(company_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_departments_branch

ON departments(branch_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_departments_parent

ON departments(parent_department_id)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 9. cost_centers (Master Data مراکز هزینه)

-- ============================================================================

CREATE TABLE cost_centers (

    cost_center_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    company_id UUID NOT NULL,

    department_id UUID,

    parent_cost_center_id UUID,

    code VARCHAR(50) NOT NULL,

    name VARCHAR(200) NOT NULL,

    description VARCHAR(500),

    manager_employee_id UUID,

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

-- Replace idx_cost_centers_department in 02_Master_Data_Table_Definitions.sql

CREATE INDEX idx_cost_centers_department

    ON cost_centers(department_id)

    WHERE deleted_at IS NULL; -- Typo Fixed (deleted at -> deleted_at)

CREATE INDEX idx_cost_centers_tenant

ON cost_centers(tenant_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_cost_centers_company

ON cost_centers(company_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_cost_centers_department

ON cost_centers(department_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_cost_centers_parent

ON cost_centers(parent_cost_center_id)

WHERE deleted_at IS NULL;

-- ============================================================================

-- 2. tenant_users (Tenant Membership)

-- ============================================================================

CREATE TABLE tenant_users (

    tenant_user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL,

    user_id UUID NOT NULL,

    employee_id UUID,

    is_owner BOOLEAN NOT NULL DEFAULT FALSE,

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_tenant_users

ON tenant_users(tenant_id, user_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_tenant_users_user

ON tenant_users(user_id)

WHERE deleted_at IS NULL;

CREATE INDEX idx_tenant_users_tenant

ON tenant_users(tenant_id)

WHERE deleted_at IS NULL;

