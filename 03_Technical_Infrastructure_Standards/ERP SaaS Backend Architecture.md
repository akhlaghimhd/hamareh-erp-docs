# ERP SaaS Backend Architecture

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Technical Infrastructure Standards
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__سند معماری Backend سیستم ERP SaaS__

__نسخه سند__

- **Version: 1.0**

__وضعیت__

Approved Architecture Guideline

__1. هدف سند__

این سند اصول و قوانین معماری Backend سیستم ERP SaaS را تعریف می‌کند تا تمامی توسعه‌های آینده بر اساس یک چارچوب مشخص، قابل توسعه، قابل نگهداری و استاندارد انجام شوند.

هدف اصلی:

- ایجاد Backend مقیاس‌پذیر برای یک سیستم ERP چندمستاجری (Multi Tenant)
- حفظ استقلال ماژول‌ها
- جلوگیری از وابستگی‌های مخرب بین بخش‌ها
- آماده‌سازی سیستم برای توسعه بلندمدت
- امکان تبدیل تدریجی به معماری Microservice در آینده در صورت نیاز

__2. تکنولوژی اصلی Backend__

__Framework__

Laravel

__Database__

PostgreSQL

__معماری ارتباطی__

API First Architecture

Backend تنها مسئول ارائه سرویس‌های Business Logic از طریق API است و هیچ ارتباط مستقیمی بین Frontend و Database وجود ندارد.

__3. سبک معماری اصلی__

معماری Backend بر اساس ترکیب اصول زیر طراحی می‌شود:

__Modular Monolith__

سیستم در ابتدا به صورت یک Backend واحد توسعه داده می‌شود، اما هر ماژول دارای مرزبندی داخلی مشخص، مستقل و قابل جداسازی خواهد بود.

هدف:

- کاهش پیچیدگی Microservice در ابتدای پروژه
- افزایش سرعت توسعه
- حفظ قابلیت جداسازی سرویس‌ها در آینده

__Domain Driven Design (DDD)__

منطق سیستم بر اساس دامنه‌های کسب‌وکار تقسیم می‌شود.

هر ماژول مسئول قوانین و رفتارهای مربوط به دامنه خود است و نباید منطق سایر دامنه‌ها را مدیریت کند.

__Clean Architecture__

ساختار Backend باید از وابستگی مستقیم Business Logic به تکنولوژی جلوگیری کند.

__4. ساختار استاندارد هر ماژول Backend__

هر ماژول باید دارای ساختار داخلی مشخص زیر باشد:

Module

├── Domain

│   ├── Entities

│   ├── Value Objects

│   ├── Domain Rules

│   └── Domain Services

│

├── Application

│   ├── Use Cases

│   ├── Application Services

│   └── DTOs

│

├── Infrastructure

│   ├── Database

│   ├── Repositories

│   └── External Services

│

└── API

    ├── Controllers

    ├── Requests

    ├── Resources

    └── Routes

__5. قانون استقلال لایه‌ها__

قوانین زیر الزامی است:

- Domain نباید وابسته به Laravel باشد.
- Domain نباید مستقیماً Database را صدا بزند.
- Controller نباید شامل Business Logic باشد.
- Repository مسئول ارتباط با Database است.
- Service مسئول اجرای فرآیندهای کسب‌وکار است.

__6. قانون API First__

تمام ارتباطات سیستم باید از طریق API انجام شود.

هیچ Frontend، Mobile Application یا سرویس خارجی اجازه دسترسی مستقیم به Database ندارد.

API باید:

- Versioned باشد.
- مستند باشد.
- دارای استاندارد ثابت Request و Response باشد.
- دارای Authentication و Authorization باشد.

__7. استاندارد Versioning API__

تمام APIها باید نسخه داشته باشند.

فرمت استاندارد:

/api/v1/

هدف:

- جلوگیری از شکستن کلاینت‌های قدیمی
- امکان توسعه نسخه‌های جدید
- استقلال توسعه ماژول‌ها

__8. مدیریت دسترسی و امنیت__

سیستم دسترسی باید فقط از Core Identity استفاده کند.

هیچ ماژولی اجازه ایجاد سیستم Authorization مستقل ندارد.

کنترل دسترسی باید بر اساس:

- User
- Role
- Permission
- Scope
- Tenant

انجام شود.

__9. قانون Multi Tenant__

سیستم بر اساس Shared Database Multi Tenant طراحی می‌شود.

قوانین:

- تمام داده‌های عملیاتی باید متعلق به یک Tenant باشند.
- تمام Queryها باید Tenant Context داشته باشند.
- هیچ سرویس یا API اجازه خواندن یا نوشتن داده بدون بررسی Tenant را ندارد.
- Data Leakage بین Tenantها خط قرمز سیستم است.

__10. قانون ارتباط بین ماژول‌ها__

ارتباط مستقیم بین ماژول‌ها ممنوع است.

قوانین:

- ماژول‌ها نباید به جداول داخلی یکدیگر وابسته باشند.
- استفاده از Foreign Key بین ماژول‌های مستقل ممنوع است.
- ارتباط باید از طریق:
	- API
	- Domain Service
	- Event
	- Logical Reference

انجام شود.

__11. Database Access Pattern__

دسترسی به Database باید از مسیر زیر باشد:

Controller

↓

Application Service

↓

Repository

↓

Database

Controller نباید Query مستقیم اجرا کند.

__12. مدیریت Transaction__

Transactionها باید در سطح Application Service مدیریت شوند.

Repository نباید مسئول تصمیم‌گیری Business Transaction باشد.

__13. Event Driven آماده‌سازی آینده__

ماژول‌ها باید به گونه‌ای طراحی شوند که امکان استفاده از Event وجود داشته باشد.

هدف:

- کاهش Coupling
- توسعه غیرهمزمان
- آماده شدن برای Microservice

__14. مدیریت خطا__

تمام APIها باید از استاندارد واحد Error Response استفاده کنند.

خطاها باید شامل:

- Error Code
- Message
- Validation Details
- Trace Identifier

باشند.

__15. Logging و Monitoring__

تمام عملیات مهم باید قابل ثبت باشند.

موارد مهم:

- تغییرات داده
- ورود کاربران
- عملیات حساس
- خطاهای سیستمی

سیستم Logging نباید وابستگی مستقیم به Business Moduleها ایجاد کند.

__16. Queue و پردازش‌های سنگین__

کارهای سنگین نباید در Request اصلی اجرا شوند.

موارد مناسب برای Queue:

- ارسال پیام
- پردازش فایل
- گزارش‌گیری
- عملیات زمان‌بر

__17. Cache Strategy__

استفاده از Cache باید کنترل شده باشد.

موارد مناسب:

- اطلاعات کم‌تغییر
- تنظیمات سیستم
- داده‌های پرتکرار

Cache نباید منبع اصلی حقیقت داده باشد.

__18. آماده‌سازی برای Microservice__

Backend فعلاً Microservice نیست.

اما قوانین زیر باید رعایت شود:

- مرز ماژول‌ها مشخص باشد.
- وابستگی‌ها کنترل شوند.
- Database Ownership مشخص باشد.
- ارتباطات قابل تبدیل به API باشند.

__19. استاندارد توسعه کد__

قوانین:

- کد باید خوانا باشد.
- Naming Convention پروژه رعایت شود.
- از Duplicate Logic جلوگیری شود.
- تست‌پذیری در طراحی لحاظ شود.
- Business Ruleها مستند باشند.

__20. اصل نهایی معماری__

هر تصمیم توسعه‌ای باید این سوال را پاسخ دهد:

"آیا این تصمیم باعث افزایش استقلال، توسعه‌پذیری و نگهداری آسان‌تر سیستم در آینده می‌شود؟"

اگر پاسخ منفی باشد، آن تصمیم نباید وارد معماری اصلی شود.

__نتیجه نهایی__

Backend سیستم ERP SaaS بر اساس:

- Laravel
- PostgreSQL
- API First
- Modular Monolith
- DDD
- Clean Architecture
- Multi Tenant Shared Database
- Event Ready Design

طراحی و توسعه خواهد شد.

این سند مرجع اصلی تمام تصمیمات معماری Backend پروژه است.

