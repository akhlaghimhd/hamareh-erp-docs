# ERP SaaS Module Architecture Map v1

- **Version:** 1.1
- **Last Updated:** 2026-08-29
- **Category:** System Blueprint & Roadmaps
- **Status:** Approved (aligned with ADD_Layer_Module_Code_Mapping_v1.0)
- **Source:** HamarehERP Architecture Documentation

---

__ERP SaaS Module Architecture Map v1.1__

__وضعیت سند__

__عنوان:__ نقشه معماری ماژول‌های ERP SaaS  
__نسخه:__ 1.1  
__هدف:__ تعریف ساختار کلان ماژول‌های سیستم، مرزبندی مسئولیت‌ها، ترتیب توسعه و وابستگی بین بخش‌ها

**شماره‌گذاری لایه‌ها:** طبق  
`ADD_Layer_Module_Code_Mapping_v1.0.md` (SSOT).

__1. هدف سند__

این سند مرجع اصلی برای تصمیم‌گیری در خصوص ایجاد، توسعه و ارتباط ماژول‌های ERP SaaS است.

تمام توسعه‌های آینده باید بر اساس این نقشه و SSOT لایه‌ها انجام شود تا:

- از ایجاد ماژول‌های موازی و تکراری جلوگیری شود.
- مرز مسئولیت هر ماژول مشخص باشد.
- وابستگی‌های بین ماژول‌ها کنترل شود.
- توسعه مرحله‌ای سیستم بدون بازطراحی اساسی امکان‌پذیر باشد.
- معماری Modular Monolith در شروع و امکان Evolution به Microservices حفظ شود.

__2. اصول حاکم بر طراحی ماژول‌ها__

__2.1 استقلال ماژول‌ها__

هر ماژول مالک داده‌های داخلی و منطق کسب‌وکار خود است. ارتباط فقط از طریق API، Service Interface، Domain Event.

__2.2 عدم Coupling مستقیم دیتابیس__

FK فیزیکی بین Bounded Contextهای مستقل ممنوع. ارتباط: Logical Reference (UUID)، API، Event.

__2.3 Tenant Isolation__

در داده‌های عملیاتی مشتری: `tenant_id` الزامی. نشت اطلاعات بین مشتریان خط قرمز است.

__3. ساختار کلان لایه‌ها (قفل‌شده 2026-08-29)__

__Layer 1: SaaS Platform Business__

کد: `SaasPlatform`

- Tenant Management
- Subscription & Billing (Plans, Subscription, Platform Invoice, Wallet, Coupon)

نام پوشه Database «SaaS Business» = alias؛ نام رسمی لایه = SaaS Platform Business.

__Layer 2: SaaS Admin__

کد: `SaasAdmin`

- Admin Users / Roles / Permissions
- Audit & Logging
- System Configuration
- Notification Center (پلتفرم)
- Support & Ticketing

__Layer 3: Partner & Affiliate__

- Partners، assignments، commissions، payouts

__Layer 4: Identity & Access Core__

کد: `IdentityCore`

- Users، Credentials، Profiles
- Tenant Membership
- Authorization: Roles، Permissions، Scopes

Organization در این لایه نیست.

__Layer 5: ERP Foundation__

- Organization (`Organization`): Companies، Branches، Departments
- Master Data (`MasterData`)
- Document Management (`DocumentManagement`)
- Workflow Engine (آینده)

__Layer 6: ERP Business Modules__

ترتیب پیشنهادی توسعه:

1. Accounting & Finance
2. Inventory & Warehouse
3. Sales & CRM / Purchase (Procurement & Sales)
4. Human Resource
5. Manufacturing
6. Project Management

__Layer 7: Extensions & Integrations__

Event bus پیشرفته، BI، AI، Mobile، Data Warehouse — آینده.

__4. زیرساخت‌های Cross Cutting__

- Event Architecture (شروع: Internal / Outbox)
- API Gateway / versioned APIs
- Cache Layer (tenant-aware)
- Reporting Layer (آینده)

__5. ترتیب اجرای پروژه__

__Phase 0__ Infrastructure & Architecture Setup

__Phase 1__ SaaS Core: Layer 1–4 (Platform Business، Admin، Partner در حد نیاز، Identity)

__Phase 2__ ERP Foundation: Layer 5

__Phase 3__ ERP Business Modules: Layer 6

__Phase 4__ Enterprise / Extensions: Layer 7

__6. قوانین تغییر این نقشه__

هیچ ماژول جدیدی بدون بررسی هدف، مرز مسئولیت، مالکیت داده، وابستگی، Tenant Isolation و API Architecture اضافه نمی‌شود.

تغییر شماره لایه فقط از طریق Amendment روی `ADD_Layer_Module_Code_Mapping_v1.0.md` مجاز است.
