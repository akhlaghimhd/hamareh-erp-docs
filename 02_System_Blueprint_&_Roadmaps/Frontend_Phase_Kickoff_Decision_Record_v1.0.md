# Frontend Phase Kickoff Decision Record v1.0

- **Version:** 1.0
- **Date:** 2026-09-10
- **Status:** Locked / Non-Negotiable
- **Category:** System Blueprint & Roadmaps
- **Related Documents:**
  - Backend_Phase_Closure_Report_L1_to_L6_v1.0.md
  - ADD_Layer_Module_Code_Mapping_v1.0.md
  - ERP SaaS Architecture Consolidated Blueprint v2.0.md
  - ERP SaaS System Architecture Blueprint.md

---

## 1. Purpose

این سند تصمیمات رسمی، قطعی و **شکست‌ناپذیر** برای شروع و اجرای فاز Frontend پلتفرم HamarehERP را ثبت می‌کند.

پس از بسته شدن رسمی فاز Backend (لایه‌های ۱ تا ۶)، این سند به‌عنوان **Single Source of Truth** برای تمام تصمیمات Frontend عمل می‌کند.

هرگونه انحراف از قوانین این سند بدون Architecture Amendment رسمی، تخلف از حاکمیت پروژه محسوب می‌شود.

---

## 2. Official Repositories (Locked)

| نوع مخزن              | آدرس رسمی                                                                 | وضعیت     |
|-----------------------|---------------------------------------------------------------------------|-----------|
| **Backend Code**      | https://github.com/akhlaghimhd/hamarehSaasErp.git                         | Active    |
| **Frontend Code**     | https://github.com/akhlaghimhd/hamarehSaasErp-Front.git                   | Active    |
| **Architecture Docs** | https://github.com/akhlaghimhd/hamareh-erp-docs.git                       | Active    |

### قوانین مخازن (Non-Negotiable)

1. مخزن Frontend **کاملاً جدا** از مخزن Backend است و باید جدا باقی بماند.
2. هیچ کدی از Frontend داخل مخزن Backend و بالعکس قرار نمی‌گیرد.
3. اسناد معماری فقط در مخزن `hamareh-erp-docs` نگهداری می‌شوند.
4. هرگونه تغییر ساختاری در این سه مخزن نیازمند تصمیم رسمی است.

---

## 3. Design Tool (Locked)

**ابزار رسمی طراحی رابط کاربری:** Figma

- تمام Design System، کامپوننت‌ها، صفحات کلیدی و تصمیمات بصری باید ابتدا در Figma ثبت شوند.
- Figma منبع حقیقت بصری (Visual Single Source of Truth) پروژه است.
- پیاده‌سازی Frontend باید با طراحی‌های تأییدشده در Figma هم‌خوان باشد.

---

## 4. Official Decisions (Non-Negotiable)

### 4.1 Repository Strategy

**Decision:** مخزن Frontend کاملاً جدا از مخزن Backend است.

**Rationale:** جداسازی مسئولیت‌ها، تاریخچه Git تمیز، CI/CD مستقل، آمادگی برای Micro-Frontend.

---

### 4.2 Platform Scope (Phase 1)

**Decision:** در فاز اول فقط **Web Application** توسعه داده می‌شود.

- توسعه React Native / Mobile در این فاز ممنوع است.
- پس از رسیدن به نسخه پایدار و قابل استفاده Web، فاز Mobile به‌صورت جداگانه آغاز خواهد شد.

---

### 4.3 Module Development Priority (Locked Order)

ترتیب توسعه ماژول‌های Frontend غیرقابل تغییر است مگر با تصمیم رسمی:

1. **Foundation**
   - Organization (Company / Branch / Department)
   - Master Data پایه
   - Identity & Access (پروفایل کاربر، نقش‌ها و دسترسی‌ها در سطح Tenant)

2. **Inventory**

3. **Procurement & Sales**

4. **Accounting** (مشاهده و گزارش‌های پایه)

5. سایر ماژول‌ها (HR، Manufacturing، Project Management، Workflow پیشرفته) بر اساس اولویت کسب‌وکار بعدی

---

### 4.4 Design System & UI Approach (Hybrid – Locked)

**Decision:** رویکرد ترکیبی (Hybrid) اجباری است.

**قوانین اجرایی:**

1. ابتدا Design System پایه در Figma تعریف می‌شود (رنگ‌ها، تایپوگرافی، Spacing، کامپوننت‌های پایه).
2. چند صفحه کلیدی در Figma طراحی می‌شوند.
3. همزمان پیاده‌سازی فنی (Next.js + Shadcn/UI + Auth + Layout) آغاز می‌گردد.
4. طراحی بقیه صفحات به‌صورت ماژول‌به‌ماژول و موازی با توسعه انجام می‌شود.
5. طراحی کامل تمام صفحات قبل از شروع کدنویسی **ممنوع** است.

**UI Library رسمی:** Shadcn/UI + Tailwind CSS

---

## 5. Technical Stack (Locked)

| حوزه                    | تکنولوژی اجباری                             |
|-------------------------|---------------------------------------------||
| Framework               | Next.js (App Router) + TypeScript           |
| Styling                 | Tailwind CSS                                |
| Component Library       | Shadcn/UI                                   |
| Server State            | TanStack Query                              |
| Client State            | Zustand                                     |
| Forms & Validation      | React Hook Form + Zod                       |
| HTTP Client             | API Client مرکزی (Axios یا Fetch wrapper)   |
| Architecture Pattern    | Core Shell + Module-based                   |

تغییر هر یک از موارد بالا بدون Architecture Amendment ممنوع است.

---

## 6. Architecture Principles (Non-Negotiable)

این اصول از اسناد معماری رسمی استخراج شده و رعایت آن‌ها **اجباری و شکست‌ناپذیر** است:

1. Frontend فقط از طریق API نسخه‌دار (`/api/v1/...`) با Backend ارتباط برقرار می‌کند.
2. هیچ Business Logic در لایه Frontend نوشته نمی‌شود.
3. امنیت واقعی فقط در Backend است. Permission در Frontend صرفاً برای کنترل نمایش UI استفاده می‌شود.
4. Tenant Context باید در تمام درخواست‌ها و stateهای مرتبط حفظ شود.
5. ساختار پوشه‌ای ماژول‌محور رعایت شود (`src/modules/{module-name}`).
6. Design System و کامپوننت‌های مشترک فقط در لایه `shared` نگهداری شوند.
7. صفحات باید Mobile-First و کاملاً Responsive باشند.
8. پشتیبانی کامل از RTL الزامی است.
9. ارتباط مستقیم Frontend با دیتابیس تحت هر شرایطی ممنوع است.

---

## 7. Local Development Policy

| موضوع                    | قانون                                      |
|--------------------------|--------------------------------------------|
| ساختار پوشه‌ها در لوکال  | دو پوشه کاملاً جدا (Backend و Frontend)    |
| اجرای Backend            | فقط از طریق Docker Compose                 |
| اجرای Frontend           | `pnpm dev` یا `npm run dev`                |
| آدرس API در توسعه        | از طریق `NEXT_PUBLIC_API_BASE_URL`         |
| CORS                     | باید در Backend برای `localhost:3000` فعال باشد |

---

## 8. Phase 0 – Mandatory Kickoff Scope

قبل از شروع هر ماژول واقعی، فاز ۰ باید تکمیل شود:

1. ایجاد و راه‌اندازی مخزن `hamarehSaasErp-Front`
2. راه‌اندازی پروژه Next.js + TypeScript + ESLint + Prettier
3. نصب و پیکربندی کامل Tech Stack قفل‌شده
4. پیاده‌سازی API Client مرکزی (Token، Refresh، Error Handling، Tenant Header)
5. پیاده‌سازی Auth Flow پایه (Login / Logout / Session)
6. پیاده‌سازی Tenant Context
7. ساخت Layout اصلی (Shell)
8. تعریف Design System پایه در Figma

**خروجی اجباری فاز ۰:** یک Shell قابل اجرا که کاربر بتواند Login کند و Tenant را بشناسد.

---

## 9. Governance & Change Control

- این سند از تاریخ ۲۰۲۶-۰۹-۱۰ **Locked** است.
- هرگونه تغییر در تصمیمات، Tech Stack، اولویت ماژول‌ها، یا سیاست مخازن نیازمند ثبت Architecture Amendment رسمی است.
- تخلف از قوانین این سند به‌عنوان نقض حاکمیت پروژه تلقی می‌شود.

---

## 10. Approval Record

| نقش              | وضعیت     | تاریخ       |
|------------------|-----------|-------------|
| Product Owner    | Approved  | 2026-09-10  |
| Technical Lead   | Approved  | 2026-09-10  |

---

**End of Document**

**This document is Non-Negotiable.**
