# 03 event outbox infrastructure

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Technical Infrastructure Standards
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- ============================================================================

-- 1. event_outbox (لجر حیاتی انتشار رویدادهای بین‌ماژولی و تضمین یکپارچگی داده‌ها)

-- ============================================================================

CREATE TABLE event_outbox (

    event_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    tenant_id UUID NOT NULL, -- ایزوله‌سازی چندمستأجری زنده رویدادها

    aggregate_type VARCHAR(100) NOT NULL, -- نام موجودیت اصلی (e.g., 'pur_orders')

    aggregate_id UUID NOT NULL, -- شناسه فیزیکی رکورد تغییر یافته

    event_type VARCHAR(150) NOT NULL, -- نوع رویداد (e.g., 'procurement.order.approved')

    payload JSONB NOT NULL, -- بدنه کامل داده‌های رویداد جهت مصرف در Event Bus

    status SMALLINT NOT NULL DEFAULT 1, -- 1: Pending, 2: Processed, 3: Failed

    retry_count INT NOT NULL DEFAULT 0,

    error_log TEXT,

    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    processed_at TIMESTAMPTZ

);

-- ایندکس ترکیبی با قید وضعیت جهت سرعت بالای پولینگِ ورکرها (High Performance Worker Polling)

CREATE INDEX idx_event_outbox_polling 

ON event_outbox (status, created_at ASC) 

WHERE status = 1;

-- ایندکس متمرکز GIN روی بدنه داده‌ها جهت کوئری‌های عارضه‌ای و رهگیری رویدادها

CREATE INDEX idx_event_outbox_payload 

ON event_outbox USING gin (payload);

