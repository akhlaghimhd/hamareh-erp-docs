# Workflow Architecture Standard

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Technical Infrastructure Standards
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__HamarehERP__

__ERP Workflow, Business Process & Configurable Manufacturing Architecture Standard v1.0__

__وضعیت سند__

- نوع سند: Architecture Standard
- پروژه: HamarehERP
- نسخه: v1.0
- وضعیت: Approved Architecture Decision
- حوزه: ERP.Core و ERP Business Modules

__1. هدف سند__

این سند اصول معماری مربوط به Workflow، Business Process Engine و Manufacturing Module در HamarehERP را تعریف می‌کند.

هدف اصلی این معماری، ایجاد یک ERP عمومی و قابل تنظیم برای صنایع مختلف است؛ به شکلی که هر Tenant بتواند فرآیندهای اختصاصی خود را بدون نیاز به توسعه نرم‌افزاری مدیریت کند.

__2. تصمیم معماری Workflow__

__2.1 جایگاه Workflow__

Workflow یک ماژول کسب‌وکاری ERP نیست.

Workflow یک قابلیت زیرساختی در ERP.Core است که توسط ماژول‌های مختلف ERP استفاده می‌شود.

جایگاه:

ERP.Core

 ├── Workflow Engine

 ├── Business Process Engine

 ├── Notification

 ├── Event System

 ├── Audit

 └── Security

__2.2 مسئولیت Workflow__

Workflow مسئول موارد زیر است:

- Approval
- Decision Making
- Process Routing
- Task Triggering
- Notification Trigger
- Escalation
- Process Control

Workflow مسئول اجرای عملیات تخصصی ماژول‌ها نیست.

__2.3 اصل جداسازی__

Workflow نباید جایگزین Business Module شود.

نمونه اشتباه:

Workflow

ثبت سفارش

    |

تولید

    |

ارسال

    |

حسابداری

نمونه صحیح:

Sales Module

      |

      ▼

Business Process

      |

      ▼

Workflow Engine

      |

      ▼

Approval / Task / Notification

__3. Business Process Engine Architecture__

__3.1 هدف__

Business Process Engine وظیفه مدیریت فرآیندهای قابل تنظیم سازمانی را دارد.

سیستم نباید فرآیندهای همه شرکت‌ها را به صورت ثابت تعریف کند.

هر Tenant باید بتواند فرآیندهای خود را بر اساس نیاز سازمان تنظیم کند.

__3.2 System Template و Tenant Workflow__

معماری Workflow دارای سه سطح است:

System Workflow Template

        |

        | Clone / Customize

        ▼

Tenant Workflow

        |

        | Execute

        ▼

Workflow Instance

__3.3 System Workflow Template__

ERP تعدادی Workflow استاندارد ارائه می‌کند.

مثال:

- تأیید فاکتور خرید
- تأیید سفارش فروش
- درخواست خرید
- درخواست مرخصی
- تأیید هزینه

Templateهای سیستمی مستقیماً توسط Tenant تغییر داده نمی‌شوند.

__3.4 Tenant Custom Workflow__

هر شرکت می‌تواند Workflow اختصاصی خود را ایجاد یا از Template موجود سفارشی‌سازی کند.

مثال:

شرکت A:

ثبت فاکتور

      ↓

مدیر مالی

      ↓

پایان

شرکت B:

ثبت فاکتور

      ↓

مدیر مالی

      ↓

ذیحساب

      ↓

مدیرعامل

      ↓

پایان

__4. Workflow Versioning Principle__

Workflow باید قابلیت نسخه‌بندی داشته باشد.

هدف:

تغییر فرآیندهای آینده نباید Workflowهای در حال اجرا را خراب کند.

نمونه:

Invoice Approval v1

Invoice Approval v2

Invoice Approval v3

هر Workflow Instance باید به نسخه مشخص خود وابسته باشد.

__5. Manufacturing Module Architecture__

__5.1 جایگاه Manufacturing__

Manufacturing یک Business Module مستقل در ERP است.

جایگاه:

ERP Modules

 ├── Sales

 ├── Inventory

 ├── Manufacturing

 ├── Accounting

 ├── CRM

 └── HR

__5.2 اصل Manufacturing عمومی__

HamarehERP برای هر صنعت یک ERP جداگانه ایجاد نمی‌کند.

یک Manufacturing Engine قابل تنظیم ارائه می‌دهد.

هر شرکت فرآیند تولید خود را با Configuration تعریف می‌کند.

__6. Manufacturing Core Components__

__6.1 Product__

تعریف محصول نهایی.

مثال:

- کتاب نوتوپیا
- پمپ کولر
- مایع شوینده

__6.2 BOM (Bill of Material)__

تعریف مواد اولیه مورد نیاز.

مثال:

محصول: پمپ کولر

مواد:

- بدنه

- موتور

- سیم

- پیچ

- بسته‌بندی

__6.3 Routing__

تعریف مراحل تولید.

مثال نوتوپیا:

طراحی

↓

چاپ

↓

جلد

↓

صحافی

↓

بسته‌بندی

↓

ارسال

__6.4 Work Center__

تعریف واحد یا مرکز کاری.

مثال:

- واحد طراحی
- خط تولید
- واحد کنترل کیفیت
- بسته‌بندی

__6.5 Production Order__

سند اجرایی تولید.

شامل:

- محصول
- مقدار تولید
- مواد مصرفی
- مراحل تولید
- وضعیت اجرا
- هزینه تولید

__7. ارتباط Manufacturing با Workflow__

Manufacturing عملیات تولید را مدیریت می‌کند.

Workflow کنترل فرآیند و تصمیم‌ها را مدیریت می‌کند.

مثال:

Production Order

        |

        ▼

Design Operation

        |

        ▼

Workflow Approval

        |

        ▼

Continue Production

__8. نمونه‌های چند صنعت__

__صنعت چاپ__

Customer Order

↓

Production Order

↓

Design

↓

Printing

↓

Binding

↓

Packaging

↓

Shipping

↓

Accounting

__صنعت پمپ کولر__

Production Order

↓

Plastic Injection

↓

Motor Assembly

↓

Quality Test

↓

Packaging

__صنعت شوینده__

Production Order

↓

Material Mixing

↓

Laboratory Control

↓

Filling

↓

Labeling

↓

Packing

__9. اصل Configurable ERP__

Tenant نباید برای ایجاد فرآیند جدید نیاز به برنامه‌نویسی داشته باشد.

Tenant باید بتواند تنظیم کند:

- محصولات
- مواد اولیه
- مراحل تولید
- مسئول هر مرحله
- واحد سازمانی
- زمان استاندارد
- کنترل کیفیت
- Workflow Approval

__10. نتیجه معماری__

معماری نهایی HamarehERP:

ERP.Core

 ├── Workflow Engine

 ├── Business Process Engine

 ├── Notification

 ├── Event

 ├── Audit

ERP Modules

 ├── Sales

 ├── Inventory

 ├── Manufacturing

 ├── Accounting

 └── Other Modules

این معماری باعث می‌شود HamarehERP بتواند صنایع مختلف را بدون تغییر Core سیستم پشتیبانی کند.

__وضعیت تصمیم__

این سند به عنوان مرجع طراحی آینده Workflow، Business Process و Manufacturing در HamarehERP در نظر گرفته می‌شود.

