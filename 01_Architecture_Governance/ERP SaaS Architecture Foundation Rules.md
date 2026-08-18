# ERP SaaS Architecture Foundation Rules

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Architecture Governance
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__Architecture Constitution__

__سند قوانین بنیادین معماری ERP SaaS__

__Version 1.1__

__1. هدف سند__

این سند به عنوان مرجع اصلی تصمیم‌گیری معماری پروژه ERP SaaS تهیه شده است.

تمامی تصمیمات مربوط به:

- طراحی دیتابیس
- ایجاد جداول
- توسعه ماژول‌ها
- طراحی API
- پیاده‌سازی سرویس‌ها
- توسعه امکانات جدید

باید با اصول این سند منطبق باشند.

هیچ توسعه‌ای نباید صرفاً بر اساس نیاز لحظه‌ای انجام شود و هر تصمیم باید جایگاه خود را در معماری کلان سیستم داشته باشد.

__2. اصل Platform First__

این محصول یک Platform است، نه مجموعه‌ای از نرم‌افزارهای مستقل.

معماری کل سیستم بر پایه:

Platform

↓

Core

↓

Modules

↓

Extensions

↓

Integrations

طراحی می‌شود.

تمام قابلیت‌های جدید باید روی این ساختار قرار بگیرند.

__3. اصل Core Only Once__

تمام مفاهیم پایه سیستم فقط یک بار و در Core تعریف می‌شوند.

نمونه:

- Tenant
- User
- Role
- Permission
- Scope
- Organization
- Company
- Branch
- Department
- Audit
- Notification
- Configuration

هیچ ماژولی اجازه ایجاد نسخه موازی از این موجودیت‌ها را ندارد.

__4. اصل Shared Core__

تمام ماژول‌ها باید از Core استفاده کنند.

منطق‌های زیر نباید در ماژول‌ها تکرار شوند:

- مدیریت کاربر
- احراز هویت
- کنترل دسترسی
- مدیریت سازمان
- Tenant Management
- Audit

__5. اصل Tenant Isolation By Design__

امنیت اطلاعات مشتریان یکی از اصول غیرقابل مذاکره سیستم است.

در معماری Shared Database Multi-Tenant:

- تمام جداول عملیاتی ماژول‌ها باید دارای tenant_id باشند.
- هیچ Query، API یا Service اجازه دسترسی بدون محدود کردن Tenant را ندارد.
- هیچ داده‌ای نباید بین Tenantها قابل مشاهده باشد.

نشت اطلاعات بین مشتریان (Data Bleeding) یک خطای بحرانی معماری محسوب می‌شود.

__6. اصل Independent Modules__

هر ماژول یک Bounded Context مستقل است.

نمونه:

- Accounting
- Inventory
- CRM
- HR
- Payroll
- Manufacturing
- Sales
- Purchase
- Asset Management

هر ماژول:

- مالک منطق کسب‌وکار خود است.
- ساختار داخلی خود را مدیریت می‌کند.
- مسئول داده‌های دامنه خود است.

__7. اصل No Direct Coupling__

وابستگی مستقیم بین Bounded Contextها ممنوع است.

اما این قانون به معنی حذف کامل Foreign Key نیست.

قانون صحیح:

__داخل یک Bounded Context:__

استفاده از Physical Foreign Key مجاز است.

مثال:

Invoice

↓

Invoice Items

↓

Invoice Payments

__بین Bounded Contextهای مستقل:__

Physical Foreign Key ممنوع است.

ارتباط باید از طریق:

- Logical Reference (UUID)
- API
- Event
- Service Contract

باشد.

هدف:

حفظ امکان جداسازی سرویس‌ها و مهاجرت آینده به معماری Microservice.

__8. اصل استاندارد ساختار هر Module__

هر ماژول باید حداقل دارای بخش‌های زیر باشد:

__1. Master Data__

اطلاعات پایه ماژول.

__2. Transactions__

عملیات روزمره.

__3. Documents__

اسناد و گردش اطلاعات.

__4. Workflow__

فرآیندهای تایید و گردش کار.

__5. Reports__

گزارش‌ها و تحلیل‌ها.

__6. Settings__

تنظیمات اختصاصی ماژول.

__7. Permission Integration__

اتصال به سیستم دسترسی Core.

__8. API / Gateway Layer__

تمام ارتباطات بیرونی ماژول باید از یک لایه مشخص API انجام شود.

__9. License & Feature Flag Check__

هر ماژول قبل از اجرا باید بررسی کند:

- آیا Tenant این ماژول را خریداری کرده؟
- آیا Feature مربوط فعال است؟
- آیا محدودیت Plan اجازه استفاده می‌دهد؟

__9. اصل Central Authorization__

هیچ ماژولی اجازه ایجاد سیستم Authorization مستقل ندارد.

تمام کنترل دسترسی باید از Core انجام شود.

ماژول‌ها فقط مصرف‌کننده خروجی Core هستند.

مبنای کنترل:

User

↓

Role

↓

Permission

↓

Scope

↓

Resource

برای مثال:

کاربر فقط در شعبه مشخص یا انبار مشخص اجازه فعالیت دارد.

__10. اصل Scope Driven Access__

سیستم Scope باید تمام محدودیت‌های سطح دسترسی را مدیریت کند.

Scope باید قابلیت توسعه برای آینده داشته باشد.

نمونه:

- Company Scope
- Branch Scope
- Warehouse Scope
- Department Scope
- Project Scope

ماژول جدید نباید سیستم Scope جدید ایجاد کند.

__11. اصل Event Driven Ready__

تمام ماژول‌ها باید قابلیت تولید و مصرف Event داشته باشند.

مثال:

Inventory:

InventoryReceiptCreated

Accounting:

AccountingDocumentCreated

ارتباط بین ماژول‌ها نباید به Query مستقیم دیتابیس وابسته باشد.

__12. اصل API و Event Versioning__

تمام APIها و Eventها باید نسخه‌بندی شوند.

مثال:

API:

/api/v1/orders

Event:

OrderCreated.v1

تغییر یک ماژول نباید باعث نیاز به تغییر همزمان سایر ماژول‌ها شود.

__13. اصل Module Independence__

هر ماژول باید بتواند مستقل توسعه، تست و ارتقا پیدا کند.

ارتقای یک ماژول نباید باعث شکستن سایر ماژول‌ها شود.

__14. اصل Backward Compatibility__

تمام توسعه‌ها باید سازگاری با نسخه‌های قبلی را حفظ کنند.

تغییرات Breaking فقط با برنامه مهاجرت رسمی انجام می‌شوند.

__15. اصل Business Rules Separation__

منطق کسب‌وکار نباید داخل Database قرار گیرد.

Database مسئول:

- ذخیره داده
- Constraint
- Index
- Integrity

است.

منطق‌هایی مانند:

- محاسبه مالیات
- پورسانت
- تخفیف
- Workflow
- قوانین تایید
- قوانین قیمت‌گذاری

باید در:

Service Layer

پیاده‌سازی شوند.

استفاده از:

- Trigger
- Stored Procedure
- Function

برای Business Logic ممنوع است مگر با تصمیم معماری رسمی.

__16. اصل Configuration Before Customization__

قبل از توسعه اختصاصی برای یک مشتری باید بررسی شود:

آیا نیاز با موارد زیر قابل حل است؟

- Configuration
- Setting
- Feature Flag
- Workflow
- Rule Engine

اولویت:

Configurable Product

بیش از

Customized Product

است.

__17. اصل Reuse Before Create__

قبل از ایجاد هر جدول یا سرویس جدید باید بررسی شود:

آیا این قابلیت قبلاً در Core یا Module دیگری وجود دارد؟

ایجاد ساختار تکراری ممنوع است.

__18. اصل Single Responsibility__

هر:

- جدول
- Service
- Module
- API

باید فقط یک مسئولیت مشخص داشته باشد.

__19. اصل Extensible Architecture__

معماری باید امکان اضافه شدن قابلیت‌های آینده را بدون تغییر Core فراهم کند.

مانند:

- AI
- BI
- Mobile
- Marketplace
- IoT
- BPM
- E-Commerce

__20. اصل Architecture Before Coding__

قبل از:

- ساخت جدول
- ایجاد API
- طراحی Module
- اضافه کردن Feature

باید جایگاه آن در معماری مشخص شود.

ابتدا معماری، سپس کدنویسی.

__نتیجه نهایی معماری__

ساختار مرجع پروژه:

Platform

↓

Core

↓

Bounded Context Modules

↓

Services / APIs

↓

Extensions

این سند مرجع نهایی تصمیمات معماری پروژه ERP SaaS است و هر توسعه آینده باید با آن تطبیق داده شود.

