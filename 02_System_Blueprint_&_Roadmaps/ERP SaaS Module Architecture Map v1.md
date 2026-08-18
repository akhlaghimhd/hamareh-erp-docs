# ERP SaaS Module Architecture Map v1

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** System Blueprint & Roadmaps
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__ERP SaaS Module Architecture Map v1.0__

__وضعیت سند__

__عنوان:__ نقشه معماری ماژول‌های ERP SaaS  
__نسخه:__ 1.0 Draft  
__هدف:__ تعریف ساختار کلان ماژول‌های سیستم، مرزبندی مسئولیت‌ها، ترتیب توسعه و وابستگی بین بخش‌ها

__1. هدف سند__

این سند به عنوان مرجع اصلی برای تصمیم‌گیری در خصوص ایجاد، توسعه و ارتباط ماژول‌های ERP SaaS استفاده می‌شود.

تمام توسعه‌های آینده باید بر اساس این نقشه انجام شود تا:

- از ایجاد ماژول‌های موازی و تکراری جلوگیری شود.
- مرز مسئولیت هر ماژول مشخص باشد.
- وابستگی‌های بین ماژول‌ها کنترل شود.
- توسعه مرحله‌ای سیستم بدون بازطراحی اساسی امکان‌پذیر باشد.
- معماری Modular Monolith در شروع و امکان Evolution به Microservices در آینده حفظ شود.

__2. اصول حاکم بر طراحی ماژول‌ها__

تمام ماژول‌های سیستم باید از قوانین معماری بالادستی پروژه پیروی کنند:

__2.1 استقلال ماژول‌ها__

هر ماژول باید:

- مالک داده‌های داخلی خود باشد.
- منطق کسب‌وکار خود را مدیریت کند.
- از دسترسی مستقیم سایر ماژول‌ها به جداول داخلی خود جلوگیری کند.

ارتباط بین ماژول‌ها فقط از طریق:

- API
- Service Interface
- Domain Event

انجام می‌شود.

__2.2 عدم Coupling مستقیم دیتابیس__

هیچ ماژولی اجازه ندارد:

- به جداول داخلی ماژول دیگر Foreign Key مستقیم ایجاد کند.
- Query مستقیم روی جداول ماژول دیگر انجام دهد.

ارتباط داده‌ای فقط از طریق:

- Logical Reference
- API
- Event

انجام می‌شود.

__2.3 Tenant Isolation__

در تمام ماژول‌هایی که داده عملیاتی مشتری را نگهداری می‌کنند:

وجود tenant_id الزامی است.

هیچ سرویس، API یا Query بدون اعمال محدودیت Tenant اجازه دسترسی به داده‌ها را ندارد.

نشت اطلاعات بین مشتریان (Data Bleeding) خط قرمز سیستم است.

__3. ساختار کلان لایه‌ها__

معماری سیستم به پنج بخش اصلی تقسیم می‌شود:

__Layer 1: SaaS Platform Layer__

__هدف:__

مدیریت خود پلتفرم SaaS و ارتباط تجاری با مشتریان.

__ماژول‌ها:__

__1. Tenant Management__

مسئول:

مدیریت مشتریان سیستم SaaS

شامل:

- Tenant Registration
- Tenant Profile
- Tenant Domain
- Tenant Status
- Tenant Configuration

وابستگی:

هیچ وابستگی به ERP ندارد.

__2. Subscription & Billing__

مسئول:

مدیریت مدل درآمدی SaaS

شامل:

- Plans
- Plan Versions
- Features
- Addons
- Subscription
- Invoice
- Payment

__3. Payment Gateway__

مسئول:

مدیریت اتصال به سرویس‌های پرداخت.

شامل:

- Payment Provider
- Transaction
- Payment Status
- Callback Handling

__4. Notification Center__

مسئول:

مرکز ارسال پیام‌های سیستم.

شامل:

- Email Notification
- SMS Notification
- In-App Notification
- Template Management
- Notification Queue

__Layer 2: Platform Administration Layer__

__هدف:__

مدیریت داخلی پلتفرم توسط تیم مالک سیستم.

__ماژول‌ها:__

__1. Admin Management__

شامل:

- Admin Users
- Admin Roles
- Admin Permissions

__2. Audit & Logging__

مسئول:

ثبت تمام فعالیت‌های حساس.

شامل:

- User Activity
- Security Events
- Data Change Tracking

__3. System Configuration__

شامل:

- Global Settings
- Environment Configuration
- Feature Configuration

__4. Support & Ticketing__

مسئول:

ارتباط پشتیبانی بین مشتری و تیم SaaS.

شامل:

- Ticket
- Conversation
- Priority
- Assignment
- Status Flow

__Layer 3: Identity & Access Core__

__هدف:__

هسته امنیت و کنترل دسترسی کل سیستم.

__ماژول‌ها:__

__1. Identity Management__

شامل:

- Users
- Credentials
- Profiles

__2. Tenant Membership__

شامل:

- Tenant Users
- User Membership

__3. Authorization__

شامل:

- Roles
- Permissions
- Scopes
- Access Rules

__Layer 4: ERP Core Foundation__

__هدف:__

ارائه زیرساخت مشترک تمام ماژول‌های ERP.

__ماژول‌ها:__

__1. Organization Management__

مسئول ساختار سازمانی مشتری.

شامل:

- Companies
- Branches
- Departments

__2. Master Data Management__

اطلاعات پایه مشترک.

شامل:

- Categories
- Units
- Currencies
- Tax Rules

__3. Document Management__

مدیریت فایل‌ها و اسناد.

شامل:

- Attachments
- Files
- Document Metadata

__4. Workflow Engine__

مدیریت فرآیندهای تایید.

شامل:

- Workflow Definition
- Approval Steps
- Execution History

__Layer 5: ERP Business Modules__

__هدف:__

ماژول‌های تخصصی کسب‌وکار.

ترتیب پیشنهادی توسعه:

__Module 1: Accounting & Finance__

اولین ماژول اصلی ERP

مسئول:

- Ledger
- Accounting Entries
- Financial Reports
- Receivable
- Payable

__Module 2: Inventory & Warehouse__

مسئول:

- Warehouse
- Stock
- Movement
- Inventory Control

__Module 3: Sales & CRM__

مسئول:

- Customer Management
- Opportunity
- Sales Process
- Orders

__Module 4: Purchase__

مسئول:

- Supplier
- Purchase Request
- Purchase Order

__Module 5: Human Resource__

مسئول:

- Employee
- Attendance
- Payroll
- HR Process

__Module 6: Manufacturing__

مسئول:

- Production
- Bill Of Material
- Work Order

__Module 7: Project Management__

مسئول:

- Project
- Task
- Resource Planning

__4. زیرساخت‌های Cross Cutting__

این بخش‌ها متعلق به یک ماژول خاص نیستند و توسط کل سیستم استفاده می‌شوند:

__Event Architecture__

وضعیت اولیه:

Backend Internal Event Bus

هدف آینده:

امکان مهاجرت به:

- RabbitMQ
- Kafka
- Cloud Event Platform

__API Gateway Layer__

مسئول:

- Authentication
- Routing
- Rate Limiting
- Security

__Cache Layer__

استفاده برای:

- Permission Cache
- Session Data
- Frequently Used Data

__Reporting Layer__

در آینده:

- Reports
- Analytics
- BI

__5. ترتیب اجرای پروژه__

__Phase 0__

Infrastructure & Architecture Setup

شامل:

- Repository
- Environment
- Docker
- CI/CD
- Base Project

__Phase 1__

SaaS Core Platform

شامل:

- Tenant
- Identity
- Authorization
- Subscription
- Billing
- Notification

__Phase 2__

ERP Foundation

شامل:

- Organization
- Master Data
- Workflow
- Document Management

__Phase 3__

ERP Business Modules

بر اساس اولویت:

1. Accounting
2. Inventory
3. Sales
4. Purchase
5. HR
6. Manufacturing
7. Project

__Phase 4__

Enterprise Features

شامل:

- Advanced Event Bus
- BI
- AI
- Mobile Application
- Data Warehouse

__6. قوانین تغییر این نقشه__

هیچ ماژول جدیدی بدون بررسی موارد زیر اضافه نمی‌شود:

- هدف ماژول
- مرز مسئولیت
- مالکیت داده
- وابستگی‌ها
- تاثیر روی Tenant Isolation
- تاثیر روی API Architecture

این سند مرجع اصلی تصمیم‌گیری برای توسعه ماژول‌های ERP SaaS خواهد بود.

