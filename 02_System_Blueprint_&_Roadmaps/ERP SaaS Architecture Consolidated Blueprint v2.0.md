# ERP SaaS Architecture Consolidated Blueprint v2.0

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** System Blueprint & Roadmaps
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__ERP SaaS Architecture Consolidated Blueprint__

__Version 2.0__

__Status:__ Architectural Decision Document (ADD)  
__State:__ Consolidated Architecture Reference  
__Project:__ HamarehERP SaaS Platform

__1. Purpose__

این سند مرجع نهایی معماری کلان پلتفرم HamarehERP SaaS است.

هدف این سند:

- حذف تناقض بین اسناد معماری 
- تعیین ساختار نهایی لایه‌ها 
- مشخص کردن مرز Platform، Core و Business Modules 
- تعیین مالکیت داده‌ها 
- تعیین اصول ارتباط بین ماژول‌ها 
- ایجاد پایه رسمی برای طراحی Database، Backend، API و Frontend 

این سند جایگزین تصمیمات پراکنده در اسناد:

- System Architecture Blueprint 
- Module Architecture Map 
- Module Boundary Definition 
- Module Dependency Map 

می‌شود.

__2. Final Architecture Model__

معماری نهایی سیستم از پنج لایه اصلی تشکیل می‌شود:

Layer 1

SaaS Platform Layer

        ↓

Layer 2

Identity & Security Core

        ↓

Layer 3

ERP Foundation Layer

        ↓

Layer 4

ERP Business Bounded Contexts

        ↓

Layer 5

Extensions & Integrations

__3. Layer 1 - SaaS Platform Layer__

مسئولیت:

مدیریت خود پلتفرم SaaS.

شامل:

__Tenant Management__

مالک:

- Tenant 
- Tenant Profile 
- Tenant Domain 
- Tenant Status 

وظایف:

- ایجاد Tenant 
- مدیریت وضعیت سرویس 
- مدیریت قرارداد SaaS 

__Subscription & Billing__

مالک:

- Plan 
- Subscription 
- Invoice 
- Payment State 

وظایف:

- مدیریت پلن‌ها 
- کنترل Feature 
- Billing 

__Platform Administration__

مالک:

- Platform Admin 
- System Configuration 
- Audit Platform 

__4. Layer 2 - Identity & Security Core__

این لایه هسته امنیت کل سیستم است.

مالک:

- User 
- Credential 
- Profile 
- Role 
- Permission 
- Scope 
- Membership 

قوانین:

هیچ Module اجازه ایجاد Identity مستقل ندارد.

مدل دسترسی:

User

 ↓

Role

 ↓

Permission

 ↓

Scope

 ↓

Resource

__5. Layer 3 - ERP Foundation Layer__

این لایه سرویس‌های مشترک ERP است.

شامل:

__Organization Management__

مالک:

- Company 
- Branch 
- Department 

__Master Data Management__

Master Data فقط شامل داده‌های عمومی و مشترک است.

مالک:

- Currency 
- Unit 
- Country 
- Language 
- Tax Definition 

داده‌های تخصصی هر ماژول در مالکیت همان ماژول باقی می‌ماند.

مثال:

Product:

مالک Inventory

Customer:

مالک Sales / CRM

Supplier:

مالک Purchase

__Workflow Engine__

مالک:

- Workflow Definition 
- Workflow Instance 
- Approval 

تمام Moduleها فقط مصرف‌کننده Workflow هستند.

__Document Management__

مالک:

- Document 
- Attachment 
- Metadata 

تمام Moduleها از طریق API استفاده می‌کنند.

__6. Layer 4 - ERP Business Bounded Contexts__

هر ماژول:

- مالک داده خود است. 
- دیتابیس منطقی مستقل دارد. 
- Business Logic خود را نگهداری می‌کند. 

ماژول‌ها:

__Accounting__

مالک:

تمام Entityهای مالی

__Inventory__

مالک:

تمام Entityهای انبار

__Sales__

مالک:

تمام Entityهای فروش

__Purchase__

مالک:

تمام Entityهای خرید

__Manufacturing__

مالک:

تمام Entityهای تولید

__HR__

مالک:

تمام Entityهای منابع انسانی

__Project Management__

مالک:

تمام Entityهای پروژه

__7. Database Architecture__

معماری دیتابیس:

Database 1

hamareh_saas_core

مالک:

Layer 1

Layer 2

--------------------------------

Database 2

hamareh_erp_tenants

مالک:

Layer 3

Layer 4

__8. Database Ownership Rules__

هر جدول فقط یک مالک دارد.

قوانین:

مجاز:

Invoice

|

Invoice Items

چون داخل یک Bounded Context هستند.

غیرمجاز:

Sales Table

FK

Inventory Table

ارتباط فقط:

- UUID Reference 
- API 
- Event 

__9. Module Communication Rules__

ارتباط مجاز:

__Synchronous__

برای Query:

- API 
- Service Contract 

__Asynchronous__

برای تغییر وضعیت:

- Domain Event 
- Integration Event 

__10. Event Architecture__

هر Event:

- یک Publisher دارد. 
- Version دارد. 
- نباید Command باشد. 

مثال:

صحیح:

InvoiceCreated.v1

غلط:

CreateInvoice

__11. Backend Architecture__

فاز اول:

Modular Monolith

\+

DDD

\+

Clean Architecture

ساختار هر Module:

Domain

Application

Infrastructure

API

Events

Repository

Authorization

Validation

__12. Frontend Architecture__

Frontend:

Next.js

ساختار:

Core Shell

\+

Independent Module Apps

هر Module مالک UI خود است.

__13. Tenant Isolation Rules__

تمام داده‌های عملیاتی:

اجباری:

tenant_id

امنیت:

- Application Filter 
- PostgreSQL RLS 

هیچ Query بدون Tenant Context مجاز نیست.

__14. Architecture Decision Rules__

قبل از ایجاد:

- Table 
- API 
- Module 
- Feature 

باید مشخص شود:

1. مالک چیست؟ 
2. محل قرارگیری چیست؟ 
3. Dependency چیست؟ 
4. Event چیست؟ 
5. اثر روی Tenant Isolation چیست؟ 

__15. Final Architecture Statement__

معماری نهایی HamarehERP:

SaaS Platform

        ↓

Identity Core

        ↓

ERP Foundation

        ↓

Business Modules

        ↓

Extensions

این سند مرجع اصلی تمام تصمیمات معماری، دیتابیس، Backend، API و Frontend خواهد بود.

هر تغییری در این ساختار نیازمند ثبت Architecture Amendment است.

