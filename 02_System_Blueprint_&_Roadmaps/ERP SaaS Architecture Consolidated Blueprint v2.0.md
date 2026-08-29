# ERP SaaS Architecture Consolidated Blueprint v2.0

- **Version:** 2.1
- **Last Updated:** 2026-08-29
- **Category:** System Blueprint & Roadmaps
- **Status:** Approved (layer model aligned with ADD_Layer_Module_Code_Mapping_v1.0)
- **Source:** HamarehERP Architecture Documentation

---

__ERP SaaS Architecture Consolidated Blueprint__

__Version 2.1__

__Status:__ Architectural Decision Document (ADD)  
__State:__ Consolidated Architecture Reference  
__Project:__ HamarehERP SaaS Platform

__1. Purpose__

این سند مرجع نهایی معماری کلان پلتفرم HamarehERP SaaS است.

هدف این سند:

- حذف تناقض بین اسناد معماری
- تعیین ساختار نهایی لایه‌ها
- مشخص کردن مرز Platform، Admin، Identity، Foundation و Business Modules
- تعیین مالکیت داده‌ها
- تعیین اصول ارتباط بین ماژول‌ها
- ایجاد پایه رسمی برای طراحی Database، Backend، API و Frontend

**شماره‌گذاری لایه‌ها و نگاشت به ماژول کد:**  
Single Source of Truth =  
`02_System_Blueprint_&_Roadmaps/ADD_Layer_Module_Code_Mapping_v1.0.md`

شماره‌گذاری قدیمی این Blueprint (Identity به‌عنوان Layer 2 و …) از تاریخ 2026-08-29 **منسوخ** است.

__2. Final Architecture Model__

```
Layer 1  SaaS Platform Business     → code: SaasPlatform
        ↓
Layer 2  SaaS Admin                 → code: SaasAdmin
        ↓
Layer 3  Partner & Affiliate
        ↓
Layer 4  Identity & Access Core     → code: IdentityCore
        ↓
Layer 5  ERP Foundation             → Organization, MasterData, DocumentManagement, Workflow
        ↓
Layer 6  ERP Business Modules       → vertical bounded contexts
        ↓
Layer 7  Extensions & Integrations  → future
```

__3. Layer 1 - SaaS Platform Business__

مسئولیت: مدیریت تجاری خود پلتفرم SaaS.

کد: `App\Modules\SaasPlatform`

شامل:

__Tenant Management__

- Tenant، Tenant Domain، Tenant Status، Tenant Settings

__Subscription & Billing__

- Plan، Plan Version، Features، Addons، Subscription، Platform Invoice، Transaction، Wallet، Coupon

توجه: «SaaS Business» در نام پوشه Database فقط **alias** است؛ نام رسمی لایه = **SaaS Platform Business**.

__4. Layer 2 - SaaS Admin__

مسئولیت: مدیریت داخلی پلتفرم توسط تیم مالک سیستم (جدا از Identity کاربران Tenant).

کد: `App\Modules\SaasAdmin`

مالک:

- Admin User / Role / Permission
- Platform Audit
- System Configuration
- Notification (پلتفرم)
- Support & Ticketing

__5. Layer 3 - Partner & Affiliate__

مسئولیت: شبکه همکاران، کمیسیون و تسویه.

جداول و دامنه طبق Database Layer 3.

__6. Layer 4 - Identity & Access Core__

هسته امنیت و کنترل دسترسی کل سیستم برای کاربران Tenant.

کد: `App\Modules\IdentityCore`

مالک:

- User، Credential، Profile
- Tenant Membership (tenant_users)
- Role، Permission، Scope و اتصالات آن‌ها

قوانین:

- هیچ Module اجازه ایجاد Identity مستقل ندارد (Core Only Once).
- مدل دسترسی: User → Role → Permission → Scope → Resource
- **Organization داخل این لایه نیست.**

__7. Layer 5 - ERP Foundation__

سرویس‌های مشترک ERP (نه عمودی کسب‌وکار).

__Organization Management__ — کد: `Organization`

- Company، Branch، Department

__Master Data Management__ — کد: `MasterData`

- Currency، Unit، Country، Tax Definition، Categories/Values، و سایر داده‌های پایه مشترک

داده تخصصی هر ماژول عمودی در مالکیت همان ماژول Layer 6 می‌ماند.

__Document Management__ — کد: `DocumentManagement`

__Workflow Engine__ — آینده؛ همه ماژول‌ها فقط مصرف‌کننده هستند.

__8. Layer 6 - ERP Business Bounded Contexts__

هر ماژول مالک داده و منطق خود است.

- Accounting
- Inventory
- Sales / Purchase (Procurement & Sales)
- Manufacturing
- HR
- Project Management

__9. Layer 7 - Extensions & Integrations__

آینده: BI، Mobile، Data Warehouse، ادغام‌های خارجی.

__10. Database Architecture__

| Logical database | Layers |
|------------------|--------|
| hamareh_saas_core | 1, 2, 3, 4 |
| hamareh_erp_tenants | 5, 6 |

__11. Database Ownership Rules__

هر جدول فقط یک مالک دارد.

مجاز: FK فیزیکی فقط داخل یک Bounded Context.

غیرمجاز: FK فیزیکی بین ماژول‌های مستقل.

ارتباط بین ماژول‌ها: UUID Reference، API، Domain/Integration Event.

__12. Module Communication Rules__

Synchronous (Query): API / Service Contract (in-process در Modular Monolith).

Asynchronous (state change): Domain Event / Integration Event.

__13. Event Architecture__

هر Event یک Publisher و Version دارد و Command نیست.

صحیح: `InvoiceCreated.v1` — غلط: `CreateInvoice`

__14. Backend Architecture__

فاز اول: Modular Monolith + DDD + Clean Architecture.

ساختار استاندارد ماژول: Domain، Application، Infrastructure، API.

__15. Frontend Architecture__

Next.js — Core Shell + Independent Module Apps.

__16. Tenant Isolation Rules__

داده‌های عملیاتی: `tenant_id` اجباری.

امنیت: Application Filter + PostgreSQL RLS.

هیچ Query بدون Tenant Context مجاز نیست.

__17. Architecture Decision Rules__

قبل از ایجاد Table / API / Module / Feature مشخص شود:

1. مالک چیست؟
2. شماره و نام لایه (طبق ADD Mapping) چیست؟
3. ماژول کد چیست؟
4. Dependency و Event چیست؟
5. اثر روی Tenant Isolation چیست؟

__18. Final Architecture Statement__

```
SaaS Platform Business (L1)
        ↓
SaaS Admin (L2)
        ↓
Partner (L3)
        ↓
Identity & Access Core (L4)
        ↓
ERP Foundation (L5)
        ↓
ERP Business Modules (L6)
        ↓
Extensions (L7)
```

این سند همراه با `ADD_Layer_Module_Code_Mapping_v1.0.md` مرجع اصلی تصمیمات معماری است.

هر تغییر در ساختار لایه‌ها نیازمند Architecture Amendment است.
