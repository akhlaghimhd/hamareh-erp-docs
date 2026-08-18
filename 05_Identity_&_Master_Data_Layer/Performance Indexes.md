# Performance Indexes

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Identity & Master Data Layer
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

-- ============================================================================

-- MODULE: ACCOUNTS & MASTER DATA PERFORMANCE INDEXES

-- ============================================================================

-- ۱. بهینه‌سازی پولینگ و جوین منطقی پرسنل به لایه هویت

CREATE INDEX idx_employees_user_lookup 

ON employees (tenant_id, user_id) 

WHERE deleted_at IS NULL;

-- ۲. بهینه‌سازی فیلتر چندشکلی پیوست‌ها بر اساس نوع موجودیت

CREATE INDEX idx_attachments_entity_type 

ON attachments (tenant_id, target_entity_type) 

WHERE deleted_at IS NULL;

-- ============================================================================

-- MODULE: PROCUREMENT & SALES LOGICAL REFERENCE INDEXES

-- ============================================================================

-- ۳. ایندکس رسید خرید روی شناسه منطقی سفارش خرید بالادستی و تامین‌کننده

CREATE INDEX idx_pur_receipts_source_order 

ON purchase_receipts (tenant_id, id_purchase_order_source, supplier_id) 

WHERE deleted_at IS NULL;

-- ۴. ایندکس حواله خروج روی شناسه سفارش فروش و مشتری

CREATE INDEX idx_sal_deliveries_source_order 

ON sales_delivery_orders (tenant_id, id_sales_order_source, customer_id) 

WHERE deleted_at IS NULL;

-- ۵. ایندکس اسناد مرجوعی روی شناسه‌های منطقی ناهمگام

CREATE INDEX idx_return_orders_source_ref 

ON return_orders (tenant_id, source_document_type, source_document_id) 

WHERE deleted_at IS NULL;

-- ============================================================================

-- MODULE: FINANCIAL ACCOUNTING ADVANCED INDEXES

-- ============================================================================

-- ۶. ایندکس تراکنش‌های مالیاتی بر اساس تاریخ جهت گزارش‌های فصلی سریع

CREATE INDEX idx_fin_tax_trans_date 

ON fin_acc_tax_transactions (tenant_id, transaction_date DESC);

-- ۷. ایندکس اموال و دارایی‌های ثابت بر اساس وضعیت دارایی

CREATE INDEX idx_fin_assets_status 

ON fin_acc_fixed_assets (tenant_id, status) 

WHERE deleted_at IS NULL;

-- ۸. ایندکس بودجه بر روی ترکیب حساب و مرکز هزینه برای کنترل آنی سقف بودجه

CREATE INDEX idx_fin_budget_items_control 

ON fin_acc_budget_items (tenant_id, account_id, cost_center_id);

