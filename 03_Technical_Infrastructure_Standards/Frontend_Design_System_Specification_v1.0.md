# Frontend Design System Specification v1.0

- **Version:** 1.1
- **Date:** 2026-09-10
- **Status:** Locked / Non-Negotiable (Structure)
- **Category:** Technical Infrastructure Standards – Frontend
- **Related Documents:**
  - Frontend_Phase_Kickoff_Decision_Record_v1.0.md
  - ERP SaaS Frontend Architecture.md
  - ERP SaaS Frontend Integration Architecture.md
  - Backend_Phase_Closure_Report_L1_to_L6_v1.0.md
- **Figma File:** https://www.figma.com/design/RykextIEfCWahPyaqEnAuZ (Hamareh ERP – Design System)

---

## 1. Purpose

این سند مشخصات کامل و رسمی Design System پلتفرم Hamareh ERP را تعریف می‌کند.

هدف:
- ایجاد مرجع واحد برای تمام تصمیمات بصری و تعاملی Frontend
- جلوگیری از پراکندگی و تصمیم‌گیری مجدد
- پشتیبانی از توسعه موازی سیاست‌گذاری و اجرا
- تضمین کیفیت سازمانی و مقیاس‌پذیری UI/UX

هرگونه انحراف از ساختار این سند بدون Architecture Amendment ممنوع است.

---

## 2. Language & Direction (Non-Negotiable)

| موضوع | تصمیم قطعی |
|------|------------|
| **زبان اصلی رابط کاربری** | فارسی |
| **جهت اصلی** | راست‌به‌چپ (RTL) |
| **فونت اصلی** | Vazirmatn |
| **زبان‌های بعدی** | عربی و انگلیسی در فازهای بعدی (در صورت نیاز) |
| **برچسب‌ها و نمونه‌متن‌ها در Figma** | باید فارسی باشند |
| **کد Frontend** | `lang="fa"` و `dir="rtl"` |

تمام کامپوننت‌ها، الگوها، پیام‌ها و مستندات بصری Design System باید بر اساس فارسی و RTL طراحی و پیاده‌سازی شوند.

---

## 3. Official Repositories & Tools

| نوع | آدرس / ابزار |
|-----|-------------|
| Backend Code | https://github.com/akhlaghimhd/hamarehSaasErp.git |
| Frontend Code | https://github.com/akhlaghimhd/hamarehSaasErp-Front.git |
| Architecture Docs | https://github.com/akhlaghimhd/hamareh-erp-docs.git |
| Design Tool (Visual SSOT) | Figma – فایل Hamareh ERP – Design System |

---

## 4. Design System Structure (Complete Inventory)

### 4.1 Foundations
- سیستم رنگ (Primary, Secondary, Success, Warning, Danger, Info + مقیاس کامل + حالت تاریک)
- تایپوگرافی (Vazirmatn – عنوان ۱ تا ۶، متن اصلی، کوچک، Caption، Label)
- فاصله‌گذاری (مقیاس ۸ نقطه‌ای + توکن‌های معنایی)
- شعاع گوشه
- سایه / Elevation
- آیکون‌ها
- توکن‌های حرکت
- Breakpointها

### 4.2 Base Components
- دکمه (اصلی، ثانویه، Outline، Ghost، مخرب) + اندازه‌ها + حالت‌ها
- دکمه آیکونی
- فیلد ورودی + برچسب + راهنما + خطا
- ناحیه متن
- انتخاب‌گر / Combobox
- چک‌باکس و رادیو
- سوئیچ
- نشان / برچسب وضعیت
- آواتار
- Tooltip
- Divider
- Skeleton

### 4.3 Form Patterns
- فیلد فرم کامل
- چیدمان فرم (تک‌ستونه و دوستونه)
- بخش فرم
- اعتبارسنجی درون‌خطی
- نشانگر اجباری بودن
- دکمه‌های اقدام فرم (ذخیره / انصراف)

### 4.4 Data Display
- جدول (کامل با هدر، ردیف، مرتب‌سازی، حالت خالی و بارگذاری)
- صفحه‌بندی
- لیست داده
- لیست توضیحات
- جفت کلید-مقدار
- نشانگر وضعیت
- حالت خالی عمومی

### 4.5 Feedback & Messaging
- هشدار (موفقیت، هشدار، خطا، اطلاعات)
- Toast / اعلان
- بنر
- پیام درون‌خطی
- نوار پیشرفت / اسپینر
- دیالوگ تأیید

### 4.6 Navigation & Layout
- نوار کناری (باز و جمع‌شده)
- هدر بالا
- مسیر صفحه (Breadcrumb)
- تب‌ها
- آیتم ناوبری عمودی
- هدر صفحه
- چیدمان محتوا

### 4.7 Overlays
- مودال / دیالوگ
- Drawer (از سمت راست – RTL)
- منوی کشویی
- Popover

### 4.8 ERP-Specific Patterns
- نشان وضعیت سند
- نمایش مبلغ و ارز
- نمایش تاریخ و زمان
- تراشه کاربر
- محدودیت دسترسی
- آیتم کارتابل
- نوار فیلتر
- نوار اقدامات گروهی

### 4.9 Common States
- پیش‌فرض، هاور، فوکوس، انتخاب‌شده، غیرفعال، بارگذاری، خطا، خالی

### 4.10 Branding & Multi-Tenant Identity
- برندینگ پلتفرم (همراه‌)
- برندینگ مستأجر (لوگوی مشتری)
- انواع لوگو
- نام و شعار محصول
- برندینگ صفحه ورود

### 4.11 User & Identity Components
- آواتار کاربر
- منوی کاربر
- تراشه کاربر
- نشان نقش
- نشانگر دسترسی
- هدر پروفایل

### 4.12 Reporting & Analytics Patterns
- هدر صفحه گزارش
- پنل فیلتر
- کارت شاخص (KPI)
- ظرف نمودار
- جدول گزارش
- انتخاب بازه تاریخ
- اقدامات خروجی

### 4.13 Complementary Patterns
- جستجوی سراسری
- راهنما
- حالت اولیه فضای کاری
- مرکز اعلان‌ها
- تایم‌لاین فعالیت
- آیتم پیوست

### 4.14 Accessibility
- حالت فوکوس یکپارچه
- کنتراست رنگ (WCAG AA)
- ناوبری صفحه‌کلید
- برچسب‌های Screen Reader
- پشتیبانی از Reduced Motion

### 4.15 Density & Layout Modes
- تراکم راحت (پیش‌فرض)
- تراکم فشرده
- عرض حداکثر محتوا
- نواحی چسبان

### 4.16 Micro-interactions & Feedback
- حالت بارگذاری دکمه
- بازخورد موفقیت
- جریان تأیید عملیات مخرب
- حالت‌های هاور و فشردن

### 4.17 Content & Microcopy Guidelines
- لحن و صدای برند (رسمی، واضح، بدون ابهام)
- برچسب دکمه‌ها (فعل‌محور و فارسی)
- متن حالت خالی (اقدام‌پذیر)
- پیام‌های خطا (مشکل + راه‌حل)
- متن‌های تأیید

### 4.18 First-time & Empty Experiences
- ورود اول / خوش‌آمدگویی
- حالت خالی ماژول
- چک‌لیست راه‌اندازی
- حالت بدون دسترسی

### 4.19 Global Behavior Patterns
- چیدمان و موقعیت Toast
- هشدار تغییرات ذخیره‌نشده
- قوانین Optimistic UI
- صفحه‌بندی (ترجیح بر اسکرول بی‌نهایت)
- مرتب‌سازی و فیلتر پیش‌فرض

### 4.20 Documentation & Governance
- بایدها و نبایدها
- راهنمای استفاده
- وضعیت کامپوننت
- نسخه‌بندی Design System

---

## 5. Implementation Priority (Phased)

### Phase A – فوری (تمرکز فعلی)
Foundations کامل + دکمه + ورودی + فیلد فرم + نشان + هشدار + جدول پایه + مودال + هدر صفحه + توکن‌های برندینگ پایه

### Phase B
نوار کناری + هدر + تب‌ها + صفحه‌بندی + حالت خالی + Drawer + انتخاب‌گر + چک‌باکس/سوئیچ + منوی کاربر + آواتار

### Phase C
نوار فیلتر، اقدامات گروهی، کارتابل، وضعیت سند، الگوهای گزارش‌گیری، حالت‌های تراکم، مستندات دسترسی‌پذیری پیشرفته، راهنمای متن‌ها

---

## 6. Key Locked Decisions

| موضوع | تصمیم |
|------|--------|
| زبان اصلی | **فارسی** |
| جهت | **RTL (راست‌به‌چپ)** |
| فونت | **Vazirmatn** |
| کتابخانه UI (کد) | Shadcn/UI + Tailwind CSS |
| تراکم | راحت (پیش‌فرض) + فشرده |
| استراتژی جدول | صفحه‌بندی |
| ابزار طراحی | Figma |
| رویکرد | Hybrid |

---

## 7. Governance

- این سند از نظر ساختار، دامنه، زبان و جهت قفل است.
- جزئیات بصری در Figma به‌عنوان Visual SSOT نگهداری می‌شود.
- تمام برچسب‌ها و نمونه‌متن‌های Figma باید فارسی باشند.
- تغییرات ساختاری نیازمند Architecture Amendment است.

---

## 8. Current Execution Status

| بخش | وضعیت |
|-----|--------|
| Foundations | شروع‌شده در Figma (در حال اصلاح به فارسی) |
| Phase A Components | در حال اجرا |
| هم‌راستایی کد (Shadcn) | در انتظار |

---

**پایان سند**

**این مشخصات، منبع واحد حقیقت برای دامنه و ساختار Design System همراه‌ ERP است.**
