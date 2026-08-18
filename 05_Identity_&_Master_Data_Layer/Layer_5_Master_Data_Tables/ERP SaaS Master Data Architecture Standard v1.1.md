# ERP SaaS Master Data Architecture Standard v1.1

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Identity & Master Data Layer
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__ERP SaaS Master Data Architecture Standard__

- **Version: 1.1  **
- **Status: Approved  **
Document Type: Architecture Standard

==================================================

__1. هدف__

==================================================

این سند معماری داده‌های پایه (Master Data) پروژه ERP SaaS را تعریف می‌کند.

هدف این سند تعیین موجودیت‌هایی است که به عنوان اطلاعات پایه در کل سیستم شناخته می‌شوند و توسط چندین ماژول به صورت مشترک مورد استفاده قرار می‌گیرند.

این سند قوانین مالکیت، استفاده، نگهداری و ارتباط Master Data را مشخص می‌کند.

==================================================

__2. تعریف Master Data__

==================================================

Master Data مجموعه‌ای از اطلاعات پایه، پایدار و قابل استفاده مجدد است که مبنای فعالیت سایر ماژول‌های ERP قرار می‌گیرد.

Master Data ماهیت عملیاتی (Transactional) ندارد.

هر Master Data باید دارای یک نسخه واحد حقیقت (Single Source of Truth) باشد.

==================================================

__3. دسته‌بندی مالکیت Master Data__

==================================================

Master Data بر اساس سطح مالکیت به دو دسته تقسیم می‌شود.

__3.1 Platform Master Data__

داده‌هایی که متعلق به کل پلتفرم هستند و بین Tenantها مشترک می‌باشند.

مثال:

- Country
- Currency
- Language
- Timezone

این داده‌ها الزاماً دارای tenant_id نیستند.

__3.2 Tenant-Owned Master Data__

داده‌هایی که متعلق به یک Tenant مشخص هستند.

مثال:

- Business Partners
- Customers
- Suppliers
- Items
- Warehouses
- Departments

تمام Tenant-Owned Master Data ها باید دارای tenant_id باشند.

==================================================

__4. اصول معماری__

==================================================

Rule-01

هر موجودیت Master Data فقط یک مالک (Owner Module) دارد.

Rule-02

هیچ ماژولی مجاز به ایجاد نسخه مستقل از Master Data متعلق به ماژول دیگر نیست.

Rule-03

تمام ماژول‌ها باید اطلاعات پایه را فقط از Owner Module دریافت کنند.

Rule-04

تمام ارتباطات بین ماژول‌ها باید از طریق API انجام شود.

Rule-05

هیچ ماژولی مجاز به تغییر مستقیم اطلاعات Master Data متعلق به ماژول دیگر نیست.

Rule-06

هر Master Data دارای شناسه یکتا (UUID) است.

Rule-07

تمام Tenant-Owned Master Data ها باید دارای tenant_id باشند.

Rule-08

تمام Master Data ها باید از استاندارد Soft Delete پیروی کنند.

Rule-09

تمام Master Data ها باید از استاندارد Audit Trail پیروی کنند.

Rule-10

هر Master Data باید نسخه واحد حقیقت (Single Source of Truth) باشد.

==================================================

__5. دسته‌بندی Master Data__

==================================================

__الف) Identity Master Data__

- Users
- User Profiles
- Roles
- Permissions
- Scopes

__ب) Organization Master Data__

- Companies
- Branches
- Departments
- Cost Centers
- Positions

__ج) Business Partner Master Data__

- Business Partners
- Business Partner Roles
- Business Partner Contacts
- Business Partner Relationships

Business Partner می‌تواند نقش‌های مختلف داشته باشد:

- Customer
- Supplier
- Contractor
- Service Provider
- Carrier

__د) Financial Master Data__

- Chart of Accounts
- Fiscal Years
- Currencies
- Exchange Rates
- Tax Definitions
- Payment Terms
- Banks
- Bank Accounts

__هـ) Inventory Master Data__

- Warehouses
- Warehouse Locations
- Units of Measure
- Item Categories
- Items
- Products
- Services
- Brands
- Product Groups

__و) CRM Master Data__

- Lead Sources
- Customer Groups
- Opportunity Types

__ز) HR Master Data__

- Employees
- Job Titles
- Employment Types
- Shifts
- Leave Types

__ح) Manufacturing Master Data__

- Bill Of Materials
- Work Centers
- Routing
- Machines

==================================================

__6. قوانین استفاده__

==================================================

Rule-01

Master Data فقط یک بار ایجاد می‌شود.

Rule-02

تمام ماژول‌ها از همان رکورد اصلی استفاده می‌کنند.

Rule-03

کپی کردن اطلاعات پایه بین ماژول‌ها ممنوع است.

Rule-04

در صورت نیاز فقط شناسه موجودیت (UUID) نگهداری می‌شود.

Rule-05

ارتباط بین ماژول‌ها فقط از طریق API انجام می‌شود.

==================================================

__7. مسئولیت مالکیت داده__

==================================================

هر Master Data دارای Owner Module مشخص است.

فقط Owner Module مجاز به:

- ایجاد
- ویرایش
- حذف
- اعتبارسنجی

اطلاعات آن موجودیت است.

سایر ماژول‌ها فقط مجاز به استفاده از آن هستند.

==================================================

__8. محدودیت‌ها__

==================================================

ایجاد Master Data تکراری ممنوع است.

ایجاد جداول مشابه در چند ماژول ممنوع است.

ویرایش مستقیم اطلاعات متعلق به ماژول دیگر ممنوع است.

Tenant-Owned Master Data نباید بدون مجوز بین Tenantها به اشتراک گذاشته شود.

==================================================

__9. نتیجه__

==================================================

Master Data هسته مشترک اطلاعات ERP است.

تمام ماژول‌های سیستم باید از یک نسخه واحد اطلاعات پایه استفاده کنند تا یکپارچگی داده‌ها، توسعه‌پذیری و نگهداری بلندمدت سیستم تضمین شود.

این استاندارد مبنای طراحی Master Data های تخصصی مانند:

- Business Partner Architecture
- Item Master Architecture
- Organization Architecture

خواهد بود.

==================================================

__Status__

ERP SaaS Master Data Architecture Standard v1.1

- **Status: Locked**

