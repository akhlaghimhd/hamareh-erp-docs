# ERP SaaS Module Dependency Map v1

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** System Blueprint & Roadmaps
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__ERP SaaS Module Dependency Map v1.0__

__وضعیت سند__

__عنوان:__ نقشه وابستگی ماژول‌های ERP SaaS  
__نسخه:__ 1.0 Draft  
__هدف:__ تعیین روابط، وابستگی‌ها، ترتیب توسعه و مرزبندی ارتباطی بین ماژول‌های سیستم

__1. هدف سند__

این سند مشخص می‌کند:

- هر ماژول چه مسئولیتی دارد.
- به چه ماژول‌هایی وابسته است.
- چه ماژول‌هایی اجازه استفاده مستقیم از آن را دارند.
- ارتباط بین ماژول‌ها از چه روشی انجام می‌شود.
- ترتیب صحیح توسعه سیستم چیست.

این سند برای جلوگیری از:

- ایجاد وابستگی‌های اشتباه
- Coupling غیرضروری
- طراحی دوباره جداول
- توسعه خارج از معماری

ایجاد شده است.

__2. قوانین اصلی وابستگی ماژول‌ها__

__اصل 1: وابستگی فقط از بالا به پایین__

ماژول‌های سطح پایین‌تر نباید به ماژول‌های سطح بالاتر وابسته شوند.

ساختار کلی:

Platform Layer

        ↓

Identity & Core Layer

        ↓

ERP Foundation Layer

        ↓

Business Modules

__اصل 2: Core همیشه مستقل است__

ماژول‌های Core نباید وابسته به ماژول‌های کسب‌وکار باشند.

مثال:

Identity نباید بداند:

- حسابداری چیست.
- انبار چیست.
- فروش چیست.

اما ماژول‌های ERP می‌توانند از Identity استفاده کنند.

__اصل 3: مالکیت داده مشخص است__

هر Entity فقط یک مالک دارد.

هیچ ماژولی اجازه ندارد Entity اصلی ماژول دیگر را مدیریت کند.

__اصل 4: ارتباط بین ماژول‌ها__

ارتباط فقط از طریق:

- API Contract
- Service Interface
- Domain Event

انجام می‌شود.

ارتباط مستقیم دیتابیس ممنوع است.

__3. نمای کلی وابستگی سیستم__

                    SaaS Platform

        Tenant Management

                |

                |

        Subscription & Billing

                |

                |

        Payment Gateway

                |

                |

        Notification Center

                    |

                    ↓

              Identity Core

        Identity Management

        Authorization

        Roles

        Permissions

        Scopes

                    |

                    ↓

              ERP Foundation

        Organization

        Master Data

        Workflow

        Document Management

                    |

                    ↓

             ERP Business Modules

        Accounting

             |

        Inventory

             |

        Sales

             |

        Purchase

             |

        HR

             |

        Manufacturing

             |

        Project Management

__4. Dependency Matrix__

__Tenant Management__

وابستگی:

ندارد

مصرف‌کنندگان:

- Identity
- Subscription
- Billing
- Notification
- ERP Core

نوع ارتباط:

API / Logical Reference

__Identity & Access__

وابستگی:

- Tenant Management

مصرف‌کنندگان:

تمام ماژول‌های سیستم

مسئول:

- Authentication
- Authorization
- User Context
- Permission Validation

__Subscription & Billing__

وابستگی:

- Tenant Management

مصرف‌کنندگان:

- Module License Check
- Feature Management

مسئول:

- Plan
- Subscription
- Billing
- Invoice

__Payment Gateway__

وابستگی:

- Billing

مصرف‌کنندگان:

- Subscription

مسئول:

- Payment Transaction
- Payment Verification
- Gateway Integration

__Notification Center__

وابستگی:

- Tenant Management
- Identity

مصرف‌کنندگان:

تمام سیستم

مسئول:

- Message Delivery
- Notification Queue
- Templates

__ERP Organization__

وابستگی:

- Tenant Management
- Identity

مصرف‌کنندگان:

تمام ERP Modules

مسئول:

- Company
- Branch
- Department

__Master Data__

وابستگی:

- ERP Organization

مصرف‌کنندگان:

- Accounting
- Inventory
- Sales
- Purchase
- Manufacturing

مسئول:

داده‌های پایه مشترک

__Workflow Engine__

وابستگی:

- Identity
- Organization

مصرف‌کنندگان:

تمام فرآیندهای نیازمند تایید

__Document Management__

وابستگی:

- Identity
- Tenant

مصرف‌کنندگان:

تمام ماژول‌ها

__Accounting__

وابستگی:

- Identity
- Organization
- Master Data

مصرف‌کنندگان:

- Sales
- Purchase
- Inventory
- Manufacturing

__Inventory__

وابستگی:

- Organization
- Master Data

مصرف‌کنندگان:

- Sales
- Purchase
- Manufacturing

__Sales__

وابستگی:

- Customer Data
- Inventory
- Accounting

__Purchase__

وابستگی:

- Supplier Data
- Inventory
- Accounting

__HR__

وابستگی:

- Identity
- Organization

__Manufacturing__

وابستگی:

- Inventory
- Accounting
- Organization

__Project Management__

وابستگی:

- Identity
- Organization

__5. ترتیب پیشنهادی توسعه__

__Phase 1: SaaS Core__

اولویت:

1. Tenant Management
2. Identity Management
3. Authorization
4. Subscription
5. Billing
6. Notification
7. Audit

دلیل:

تمام سیستم به این بخش وابسته است.

__Phase 2: ERP Foundation__

اولویت:

1. Organization
2. Master Data
3. Document Management
4. Workflow

__Phase 3: اولین ماژول‌های ERP__

اولویت پیشنهادی:

__1. Accounting__

دلیل:

هسته مالی اکثر فرآیندها است.

__2. Inventory__

دلیل:

وابستگی زیاد به عملیات.

__3. Sales__

دلیل:

ایجاد گردش درآمد.

__4. Purchase__

__5. HR__

__6. Manufacturing__

__7. Project Management__

__6. ارتباط API و Event__

__ارتباطات عمومی__

برای عملیات درخواست/پاسخ:

استفاده از API

مثال:

- دریافت اطلاعات کاربر
- بررسی دسترسی
- دریافت وضعیت Subscription

__ارتباطات غیرهمزمان__

برای اتفاقات سیستمی:

استفاده از Event

مثال:

- ایجاد Tenant جدید
- فعال شدن Subscription
- ایجاد Invoice
- ثبت سفارش

__7. قوانین Event__

__اصل 1__

Event نباید جایگزین API شود.

__اصل 2__

Event باید درباره اتفاق رخ داده باشد، نه دستور اجرایی.

__اصل 3__

مصرف‌کننده Event نباید به دیتابیس تولیدکننده Event دسترسی داشته باشد.

__8. Dependency Restrictions__

موارد ممنوع:

- Business Module وابسته به Business Module دیگر به شکل مستقیم
- Foreign Key بین ماژول‌ها
- Query مستقیم روی دیتابیس ماژول دیگر
- انتقال Logic کسب‌وکار به Core

__9. مسیر تکامل آینده__

معماری فعلی:

Modular Monolith

هدف آینده:

امکان تبدیل تدریجی به:

Service Oriented Architecture

یا

Microservices

بدون بازنویسی کامل سیستم.

__10. نتیجه نهایی__

این سند تعیین می‌کند:

- چه چیزی کجا قرار می‌گیرد.
- چه چیزی به چه چیزی وابسته است.
- چه چیزی نباید وابسته شود.
- ترتیب منطقی توسعه چیست.

تمام طراحی‌های آینده باید قبل از اجرا با این نقشه تطبیق داده شوند.

