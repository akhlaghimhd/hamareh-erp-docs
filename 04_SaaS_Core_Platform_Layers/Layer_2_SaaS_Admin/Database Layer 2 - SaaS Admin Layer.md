# Database Layer 2 - SaaS Admin Layer

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** SaaS Core Platform Layers
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- ==========================================

-- Layer 2: SaaS Admin Layer (Optimized)

-- ==========================================

-- 1. admin_users

CREATE TABLE admin_users (

    admin_user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    username VARCHAR(100) NOT NULL,

    email VARCHAR(200) NOT NULL,

    password_hash TEXT NOT NULL,

    first_name VARCHAR(100),

    last_name VARCHAR(100),

    mobile VARCHAR(20),

    status SMALLINT NOT NULL DEFAULT 1,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_admin_users_username ON admin_users(username) WHERE deleted_at IS NULL;

CREATE UNIQUE INDEX uq_admin_users_email ON admin_users(email) WHERE deleted_at IS NULL;

-- 2. admin_roles

CREATE TABLE admin_roles (

    admin_role_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    code VARCHAR(50) NOT NULL,

    name VARCHAR(100) NOT NULL,

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

CREATE UNIQUE INDEX uq_admin_roles_code ON admin_roles(code) WHERE deleted_at IS NULL;

-- 3. admin_permissions

CREATE TABLE admin_permissions (

    admin_permission_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    code VARCHAR(100) NOT NULL,

    name VARCHAR(150) NOT NULL,

    module_name VARCHAR(100),

    description VARCHAR(500),

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_admin_permissions_code ON admin_permissions(code) WHERE deleted_at IS NULL;

-- 4. admin_user_roles

CREATE TABLE admin_user_roles (

    admin_user_role_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    admin_user_id UUID NOT NULL REFERENCES admin_users(admin_user_id) ON DELETE CASCADE,

    admin_role_id UUID NOT NULL REFERENCES admin_roles(admin_role_id) ON DELETE CASCADE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_admin_user_role ON admin_user_roles(admin_user_id, admin_role_id) WHERE deleted_at IS NULL;

-- 5. admin_role_permissions

CREATE TABLE admin_role_permissions (

    admin_role_permission_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    admin_role_id UUID NOT NULL REFERENCES admin_roles(admin_role_id) ON DELETE CASCADE,

    admin_permission_id UUID NOT NULL REFERENCES admin_permissions(admin_permission_id) ON DELETE CASCADE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_admin_role_permission ON admin_role_permissions(admin_role_id, admin_permission_id) WHERE deleted_at IS NULL;

-- ============================================================================

-- audit_logs

-- Shared ERP Audit Trail

-- Cross Platform Audit Logging

-- ============================================================================

CREATE TABLE audit_logs (

    audit_log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID,

    user_id UUID,

    -- Logical Reference to Layer 4 Identity User

    admin_user_id UUID,

    -- Logical Reference to SaaS Admin User

    session_id UUID,

    request_id UUID,

    entity_name VARCHAR(100) NOT NULL,

    entity_id UUID,

    action_type VARCHAR(50) NOT NULL,

    /*

        CREATE

        UPDATE

        DELETE

        LOGIN

        LOGOUT

        APPROVE

        REJECT

        EXPORT

    */

    severity SMALLINT NOT NULL DEFAULT 1,

    /*

        1 Info

        2 Warning

        3 Critical

    */

    ip_address VARCHAR(45),

    user_agent TEXT,

    old_values JSONB,

    new_values JSONB,

    details JSONB,

    log_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID

);

CREATE INDEX idx_audit_logs_tenant

ON audit_logs(tenant_id);

CREATE INDEX idx_audit_logs_entity

ON audit_logs(entity_name, entity_id);

CREATE INDEX idx_audit_logs_user

ON audit_logs(user_id);

CREATE INDEX idx_audit_logs_date

ON audit_logs(log_date DESC);

CREATE INDEX idx_audit_logs_request

ON audit_logs(request_id);

-- 7. system_settings

CREATE TABLE system_settings (

    system_setting_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    setting_key VARCHAR(150) NOT NULL,

    setting_value TEXT,

    description VARCHAR(500),

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_system_settings_key ON system_settings(setting_key) WHERE deleted_at IS NULL;

-- 8. notifications

-- Replace notifications table definition in Database Layer 2 - SaaS Admin Layer.sql

CREATE TABLE notifications (

    notification_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE RESTRICT,

    recipient_user_id UUID NOT NULL, -- Pure Logical Reference to identity.users

    title VARCHAR(200) NOT NULL,

    body TEXT NOT NULL,

    type_code VARCHAR(50) NOT NULL,

    is_read BOOLEAN NOT NULL DEFAULT FALSE,

    read_at TIMESTAMPTZ,

    

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE INDEX idx_notifications_recipient 

    ON notifications(tenant_id, recipient_user_id, is_read) 

    WHERE deleted_at IS NULL;

-- 9. support_tickets

CREATE TABLE support_tickets (

    ticket_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL REFERENCES tenants(tenant_id) ON DELETE CASCADE,

    tenant_user_id UUID, -- Logical ref to Layer 4

    assigned_admin_user_id UUID REFERENCES admin_users(admin_user_id) ON DELETE SET NULL,

    ticket_number VARCHAR(50) NOT NULL,

    subject VARCHAR(300) NOT NULL,

    description TEXT,

    priority SMALLINT NOT NULL DEFAULT 2,

    status SMALLINT NOT NULL DEFAULT 1,

    closed_at TIMESTAMPTZ,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    created_by UUID,

    updated_at TIMESTAMPTZ,

    updated_by UUID,

    deleted_at TIMESTAMPTZ,

    deleted_by UUID,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_support_tickets_number ON support_tickets(ticket_number) WHERE deleted_at IS NULL;

CREATE INDEX idx_support_tickets_tenant_user ON support_tickets(tenant_user_id);

-- ============================================================================

-- 1. admin_user_sessions (مدیریت نشست‌های فعال کاربران ارشد)

-- ============================================================================

CREATE TABLE admin_user_sessions (

    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    admin_user_id UUID NOT NULL, -- ارجاع منطقی به ادمین سیستم

    token_hash VARCHAR(256) NOT NULL,

    ip_address VARCHAR(45) NOT NULL,

    user_agent VARCHAR(500),

    is_active BOOLEAN NOT NULL DEFAULT TRUE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    expires_at TIMESTAMPTZ NOT NULL,

    last_activity_at TIMESTAMPTZ NOT NULL DEFAULT NOW()

);

CREATE INDEX idx_admin_sessions_user ON admin_user_sessions(admin_user_id) WHERE is_active = TRUE;

-- ============================================================================

-- 2. admin_login_attempts (لاگ تلاش‌های ورود جهت پایش حملات بروت‌فورس)

-- ============================================================================

CREATE TABLE admin_login_attempts (

    attempt_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    username VARCHAR(150) NOT NULL,

    ip_address VARCHAR(45) NOT NULL,

    user_agent VARCHAR(500),

    is_successful BOOLEAN NOT NULL,

    failure_reason VARCHAR(100),

    attempted_at TIMESTAMPTZ NOT NULL DEFAULT NOW()

);

CREATE INDEX idx_admin_login_failures ON admin_login_attempts(ip_address, attempted_at) WHERE is_successful = FALSE;

-- ============================================================================

-- 3. admin_api_keys (مدیریت کلیدهای دسترسی API برای ادمین‌ها)

-- ============================================================================

CREATE TABLE admin_api_keys (

    api_key_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    admin_user_id UUID NOT NULL,

    name VARCHAR(100) NOT NULL,

    key_prefix VARCHAR(10) NOT NULL,

    key_hash VARCHAR(256) NOT NULL,

    is_active BOOLEAN NOT NULL DEFAULT TRUE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    expires_at TIMESTAMPTZ,

    last_used_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_admin_api_keys_hash ON admin_api_keys(key_hash) WHERE is_active = TRUE;

-- ============================================================================

-- 4. admin_webhooks (مدیریت وب‌هوک‌های خروجی ادمین سیستم)

-- ============================================================================

CREATE TABLE admin_webhooks (

    webhook_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    name VARCHAR(150) NOT NULL,

    target_url VARCHAR(1000) NOT NULL,

    secret_token VARCHAR(256),

    event_types VARCHAR(50)[] NOT NULL, -- آرایه‌ای از رویدادها

    is_active BOOLEAN NOT NULL DEFAULT TRUE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

-- ============================================================================

-- 5. notification_templates (قالب‌های داینامیک پیام‌ها و اعلان‌ها)

-- ============================================================================

CREATE TABLE notification_templates (

    template_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    template_code VARCHAR(100) NOT NULL,

    title VARCHAR(200) NOT NULL,

    body_template TEXT NOT NULL, -- فرمت نیتیو یا مارک‌داون شامل متغیرها

    channel VARCHAR(50) NOT NULL, -- SMS, EMAIL, PUSH, IN_APP

    is_active BOOLEAN NOT NULL DEFAULT TRUE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    updated_at TIMESTAMPTZ,

    deleted_at TIMESTAMPTZ,

    row_version BIGINT NOT NULL DEFAULT 1

);

CREATE UNIQUE INDEX uq_notification_templates_code ON notification_templates(template_code) WHERE deleted_at IS NULL;

-- ============================================================================

-- 6. notification_deliveries (وضعیت و لاگ ارسال هر اعلان به تفکیک کانال)

-- ============================================================================

CREATE TABLE notification_deliveries (

    delivery_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    notification_id UUID NOT NULL, -- ارجاع منطقی به جدول اصلی اعلان‌ها

    channel VARCHAR(50) NOT NULL,

    status SMALLINT NOT NULL DEFAULT 1, -- 1: Pending, 2: Sent, 3: Failed, 4: Delivered

    error_message TEXT,

    sent_at TIMESTAMPTZ,

    delivered_at TIMESTAMPTZ,

    retry_count INT NOT NULL DEFAULT 0

);

CREATE INDEX idx_notification_deliveries_status ON notification_deliveries(status) WHERE status = 3;

-- ============================================================================

-- 7. support_ticket_messages (ریز مکالمات و پیام‌های تیکت‌های پشتیبانی)

-- ============================================================================

CREATE TABLE support_ticket_messages (

    message_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    ticket_id UUID NOT NULL, -- فیزیکی به جدول اصلی تیکت‌ها وصل شود

    sender_type SMALLINT NOT NULL, -- 1: Tenant User, 2: Admin Support

    sender_user_id UUID NOT NULL, -- ارجاع منطقی به لایه هویت

    message_body TEXT NOT NULL,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()

);

CREATE INDEX idx_support_messages_ticket ON support_ticket_messages(ticket_id);

-- ============================================================================

-- 8. support_ticket_attachments (پیوست‌های متصل به تیکت‌های پشتیبانی)

-- ============================================================================

CREATE TABLE support_ticket_attachments (

    attachment_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    message_id UUID NOT NULL REFERENCES support_ticket_messages(message_id) ON DELETE CASCADE,

    file_name VARCHAR(255) NOT NULL,

    storage_path VARCHAR(1000) NOT NULL, -- آدرس فایل در ریپازیتوری یا S3 پلتفرم

    file_size_bytes BIGINT NOT NULL,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()

);

-- ۱. اصلاح جدول کاربران مدیر سیستم

ALTER TABLE admin_users 

ADD COLUMN last_login_at TIMESTAMPTZ,

ADD COLUMN failed_login_count INT NOT NULL DEFAULT 0,

ADD COLUMN locked_until TIMESTAMPTZ,

ADD COLUMN two_factor_enabled BOOLEAN NOT NULL DEFAULT FALSE;

-- ۲. اصلاح جدول مجوزهای ادمین

ALTER TABLE admin_permissions 

ADD COLUMN permission_group VARCHAR(100),

ADD COLUMN action_type VARCHAR(50); -- C, R, U, D, EXECUTE

-- ۳. اصلاح جدول گزارش حسابرسی سیستم

ALTER TABLE audit_logs 

ADD COLUMN request_id UUID,

ADD COLUMN session_id UUID,

ADD COLUMN severity SMALLINT NOT NULL DEFAULT 1; -- 1: Info, 2: Warning, 3: Critical

-- ۴. اصلاح جدول اعلان‌ها (حذف فیزیکی FK به لایه کاربر مستأجر به دلیل ایزوله‌سازی کانتکست‌ها)

ALTER TABLE notifications DROP COLUMN IF EXISTS id_tenant_user;

ALTER TABLE notifications ADD COLUMN recipient_user_id UUID NOT NULL; -- ارجاع کاملاً منطقی

-- ۵. اصلاح جدول تیکت‌های پشتیبانی

ALTER TABLE support_tickets 

ADD COLUMN category_id UUID,

ADD COLUMN channel VARCHAR(50) NOT NULL DEFAULT 'PORTAL', -- EMAIL, PORTAL, CHAT

ADD COLUMN resolved_at TIMESTAMPTZ;

