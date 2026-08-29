# Database Layer 1 - SaaS Platform Business Layer

- **Version:** 1.1
- **Last Updated:** 2026-08-29
- **Category:** SaaS Core Platform Layers
- **Status:** Approved
- **Official layer name:** SaaS Platform Business
- **Code module:** `App\Modules\SaasPlatform`
- **Source:** HamarehERP Architecture Documentation
- **SSOT:** `ADD_Layer_Module_Code_Mapping_v1.0.md`

> Former file/folder names «SaaS Business» / `Layer_1_SaaS_Business` are retired.

---

-- =========================================================================

-- Layer 1: SaaS Platform Business Layer (REVISED & SECURED) - PART 1

-- =========================================================================

CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- 1. tenants

CREATE TABLE tenants (

 tenant_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_code VARCHAR(100) NOT NULL,

 tenant_name VARCHAR(200) NOT NULL,

 legal_name VARCHAR(300),

 tenant_type SMALLINT NOT NULL DEFAULT 1,

 slug VARCHAR(100) NOT NULL,

 primary_domain_enabled BOOLEAN NOT NULL DEFAULT FALSE,

 domain_status SMALLINT NOT NULL DEFAULT 1,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_tenants_code ON tenants(tenant_code) WHERE deleted_at IS NULL;

CREATE UNIQUE INDEX uq_tenants_slug ON tenants(slug) WHERE deleted_at IS NULL;

-- 2. tenant_wallets

CREATE TABLE tenant_wallets (

 wallet_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE RESTRICT,

 balance NUMERIC(20,4) NOT NULL DEFAULT 0,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_tenant_wallet ON tenant_wallets(tenant_id) WHERE deleted_at IS NULL;

-- 3. plans

CREATE TABLE plans (

 plan_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 code VARCHAR(50) NOT NULL,

 name VARCHAR(200) NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_plans_code ON plans(code) WHERE deleted_at IS NULL;

-- 4. plan_versions

CREATE TABLE plan_versions (

 plan_version_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 plan_id UUID NOT NULL REFERENCES plans(plan_id) ON DELETE RESTRICT,

 version_number INT NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_plan_version ON plan_versions(plan_id, version_number) WHERE deleted_at IS NULL;

-- 5. plan_prices

CREATE TABLE plan_prices (

 plan_price_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 plan_version_id UUID NOT NULL REFERENCES plan_versions(plan_version_id) ON DELETE RESTRICT,

 amount NUMERIC(20,4) NOT NULL,

 billing_period_days INT NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_plan_prices_version ON plan_prices(plan_version_id) WHERE deleted_at IS NULL;

-- 6. plan_modules

CREATE TABLE plan_modules (

 plan_module_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 plan_version_id UUID NOT NULL REFERENCES plan_versions(plan_version_id) ON DELETE RESTRICT,

 code VARCHAR(50) NOT NULL,

 name VARCHAR(200) NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_plan_modules_version ON plan_modules(plan_version_id) WHERE deleted_at IS NULL;

-- 7. plan_features

CREATE TABLE plan_features (

 plan_feature_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 plan_module_id UUID NOT NULL REFERENCES plan_modules(plan_module_id) ON DELETE RESTRICT,

 code VARCHAR(50) NOT NULL,

 name VARCHAR(200) NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_plan_features_module ON plan_features(plan_module_id) WHERE deleted_at IS NULL;

-- 8. plan_version_features

CREATE TABLE plan_version_features (

 plan_version_feature_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 plan_version_id UUID NOT NULL REFERENCES plan_versions(plan_version_id) ON DELETE RESTRICT,

 plan_feature_id UUID NOT NULL REFERENCES plan_features(plan_feature_id) ON DELETE RESTRICT,

 enabled BOOLEAN NOT NULL DEFAULT TRUE,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_plan_version_feature ON plan_version_features(plan_version_id, plan_feature_id) WHERE deleted_at IS NULL;

-- 9. plan_offers

CREATE TABLE plan_offers (

 plan_offer_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 plan_version_id UUID NOT NULL REFERENCES plan_versions(plan_version_id) ON DELETE RESTRICT,

 name VARCHAR(200) NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 start_date TIMESTAMPTZ,

 end_date TIMESTAMPTZ,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_plan_offers_version ON plan_offers(plan_version_id) WHERE deleted_at IS NULL;

-- 10. plan_offer_discounts

CREATE TABLE plan_offer_discounts (

 plan_offer_discount_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 plan_offer_id UUID NOT NULL REFERENCES plan_offers(plan_offer_id) ON DELETE RESTRICT,

 discount_value NUMERIC(20,4) NOT NULL,

 discount_type SMALLINT NOT NULL,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_plan_offer_discounts_offer ON plan_offer_discounts(plan_offer_id) WHERE deleted_at IS NULL;

-- 11. addons

CREATE TABLE addons (

 addon_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 code VARCHAR(50) NOT NULL,

 name VARCHAR(200) NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_addons_code ON addons(code) WHERE deleted_at IS NULL;

-- 12. offer_available_addons

CREATE TABLE offer_available_addons (

 offer_available_addon_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 plan_offer_id UUID NOT NULL REFERENCES plan_offers(plan_offer_id) ON DELETE RESTRICT,

 addon_id UUID NOT NULL REFERENCES addons(addon_id) ON DELETE RESTRICT,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_offer_addon ON offer_available_addons(plan_offer_id, addon_id) WHERE deleted_at IS NULL;

-- =========================================================================

-- Layer 1: SaaS Platform Business Layer - PART 2

-- =========================================================================

-- 13. subscriptions

CREATE TABLE subscriptions (

 subscription_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE RESTRICT,

 plan_version_id UUID NOT NULL REFERENCES plan_versions(plan_version_id) ON DELETE RESTRICT,

 status SMALLINT NOT NULL DEFAULT 1,

 start_date TIMESTAMPTZ,

 end_date TIMESTAMPTZ,

 next_billing_date TIMESTAMPTZ,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_subscriptions_tenant ON subscriptions(tenant_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_subscriptions_version ON subscriptions(plan_version_id) WHERE deleted_at IS NULL;

-- 14. subscription_events

CREATE TABLE subscription_events (

 subscription_event_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 subscription_id UUID NOT NULL REFERENCES subscriptions(subscription_id) ON DELETE RESTRICT,

 event_type SMALLINT NOT NULL,

 description VARCHAR(500),

 event_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_subscription_events_sub ON subscription_events(subscription_id) WHERE deleted_at IS NULL;

-- 15. subscription_addons

CREATE TABLE subscription_addons (

 subscription_addon_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 subscription_id UUID NOT NULL REFERENCES subscriptions(subscription_id) ON DELETE RESTRICT,

 addon_id UUID NOT NULL REFERENCES addons(addon_id) ON DELETE RESTRICT,

 amount NUMERIC(20,4) NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_subscription_addons_sub ON subscription_addons(subscription_id) WHERE deleted_at IS NULL;

-- 16. invoice_profiles

CREATE TABLE invoice_profiles (

 invoice_profile_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE RESTRICT,

 billing_partner_id UUID NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_invoice_profiles_tenant ON invoice_profiles(tenant_id) WHERE deleted_at IS NULL;

-- 17. platform_invoices

CREATE TABLE platform_invoices (

 invoice_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE RESTRICT,

 invoice_profile_id UUID REFERENCES invoice_profiles(invoice_profile_id) ON DELETE SET NULL,

 invoice_number VARCHAR(100) NOT NULL,

 total_amount NUMERIC(20,4) NOT NULL,

 discount_amount NUMERIC(20,4) NOT NULL DEFAULT 0,

 tax_amount NUMERIC(20,4) NOT NULL DEFAULT 0,

 final_amount NUMERIC(20,4) NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 issue_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 due_date TIMESTAMPTZ,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_platform_invoices_number ON platform_invoices(invoice_number) WHERE deleted_at IS NULL;

CREATE INDEX idx_platform_invoices_tenant ON platform_invoices(tenant_id) WHERE deleted_at IS NULL;

-- 18. platform_invoice_items

CREATE TABLE platform_invoice_items (

 invoice_item_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 invoice_id UUID NOT NULL REFERENCES platform_invoices(invoice_id) ON DELETE RESTRICT,

 item_type VARCHAR(50) NOT NULL,

 reference_id UUID,

 description VARCHAR(500),

 amount NUMERIC(20,4) NOT NULL,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_platform_invoice_items_invoice ON platform_invoice_items(invoice_id) WHERE deleted_at IS NULL;

-- 19. platform_transactions

CREATE TABLE platform_transactions (

 transaction_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 invoice_id UUID NOT NULL REFERENCES platform_invoices(invoice_id) ON DELETE RESTRICT,

 gateway VARCHAR(100),

 transaction_number VARCHAR(200),

 amount NUMERIC(20,4) NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 transaction_date TIMESTAMPTZ,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_platform_transactions_invoice ON platform_transactions(invoice_id) WHERE deleted_at IS NULL;

-- 20. tenant_wallet_transactions

CREATE TABLE tenant_wallet_transactions (

 wallet_transaction_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 wallet_id UUID NOT NULL REFERENCES tenant_wallets(wallet_id) ON DELETE RESTRICT,

 transaction_type SMALLINT NOT NULL,

 amount NUMERIC(20,4) NOT NULL,

 balance_after NUMERIC(20,4) NOT NULL,

 description VARCHAR(500),

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_tenant_wallet_tx_wallet ON tenant_wallet_transactions(wallet_id) WHERE deleted_at IS NULL;

-- 21. coupons

CREATE TABLE coupons (

 coupon_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 code VARCHAR(100) NOT NULL,

 discount_type SMALLINT NOT NULL,

 discount_value NUMERIC(20,4) NOT NULL,

 status SMALLINT NOT NULL DEFAULT 1,

 start_date TIMESTAMPTZ,

 end_date TIMESTAMPTZ,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_coupons_code ON coupons(code) WHERE deleted_at IS NULL;

-- 22. coupon_usages

CREATE TABLE coupon_usages (

 coupon_usage_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 coupon_id UUID NOT NULL REFERENCES coupons(coupon_id) ON DELETE RESTRICT,

 tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE RESTRICT,

 invoice_id UUID REFERENCES platform_invoices(invoice_id) ON DELETE SET NULL,

 discount_amount NUMERIC(20,4) NOT NULL,

 used_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_coupon_usages_tenant ON coupon_usages(tenant_id) WHERE deleted_at IS NULL;

CREATE INDEX idx_coupon_usages_coupon ON coupon_usages(coupon_id) WHERE deleted_at IS NULL;

-- 23. tenant_domains

CREATE TABLE tenant_domains (

 domain_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

 tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE RESTRICT,

 domain_name VARCHAR(255) NOT NULL,

 domain_type SMALLINT NOT NULL DEFAULT 1,

 is_primary BOOLEAN NOT NULL DEFAULT FALSE,

 verification_token VARCHAR(255),

 verified_at TIMESTAMPTZ,

 ssl_status SMALLINT NOT NULL DEFAULT 0,

 status SMALLINT NOT NULL DEFAULT 1,

 created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

 created_by UUID,

 updated_at TIMESTAMPTZ,

 updated_by UUID,

 deleted_at TIMESTAMPTZ,

 deleted_by UUID,

 row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_tenant_domains_name ON tenant_domains(domain_name) WHERE deleted_at IS NULL;

CREATE INDEX idx_tenant_domains_tenant ON tenant_domains(tenant_id) WHERE deleted_at IS NULL;
