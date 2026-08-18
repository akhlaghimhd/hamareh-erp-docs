# End of Database Ownership Matrix v1.1

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Identity & Master Data Layer
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__Database Ownership Matrix__

__Version 1.1__

__Status:__ Architectural Decision Document  
__State:__ Database Ownership Reference  
__Project:__ HamarehERP SaaS Platform

__Change Log:__

- Version 1.0 → Version 1.1 
- تکمیل مالکیت Entityهای جدید بر اساس Module Architecture Map 
- رفع ابهام مالکیت جداول مشترک بین Moduleها 
- اضافه شدن Entity Resolution برای جلوگیری از Duplicate Entity 
- هماهنگ‌سازی با Manufacturing، Financial Accounting، Procurement/Sales و Master Data Design 

__1. Purpose__

این سند مالکیت رسمی داده‌ها در پلتفرم HamarehERP را مشخص می‌کند.

قوانین:

- هر Table فقط یک Owner دارد. 
- هر Entity فقط در یک Context تعریف می‌شود. 
- هیچ Module اجازه ایجاد نسخه موازی از Entity دیگر ندارد. 
- طراحی ERD باید بر اساس این سند انجام شود. 
- Migrationها باید مطابق Ownership Matrix ایجاد شوند. 
- Reference بین Moduleها فقط از طریق Logical Reference، API یا Domain Event انجام می‌شود. 

__2. Ownership Model__

ساختار مالکیت:

SaaS Platform

        ↓

Identity Core

        ↓

ERP Foundation

        ↓

ERP Master Data

        ↓

Business Modules

__3. Database Ownership__

__3.1 SaaS Platform Database__

Database:

hamareh_saas_core

__4. SaaS Platform Layer Ownership__

__4.1 Tenant Management Module__

Schema:

saas

Owner:

Tenant Management

__Table__

__Owner__

__Description__

tenants

Tenant Management

مشتری SaaS

tenant_domains

Tenant Management

دامنه‌های مشتری

tenant_settings

Tenant Management

تنظیمات Tenant

tenant_status_history

Tenant Management

تاریخچه وضعیت Tenant

__4.2 Subscription & Billing Module__

Schema:

billing

Owner:

Subscription

__Table__

__Owner__

__Description__

plans

Subscription

پلن‌ها

plan_versions

Subscription

نسخه‌های پلن

plan_features

Subscription

امکانات پلن

addons

Subscription

افزونه‌ها

subscriptions

Subscription

اشتراک مشتری

invoices

Subscription

صورتحساب SaaS

billing_transactions

Subscription

تراکنش مالی SaaS

__4.3 Payment Gateway Module__

Schema:

payment

Owner:

Payment Gateway

__Table__

__Owner__

__Description__

payment_providers

Payment Gateway

سرویس‌دهندگان پرداخت

payment_transactions

Payment Gateway

تراکنش پرداخت

payment_callbacks

Payment Gateway

Callback پرداخت

payment_status_history

Payment Gateway

تاریخچه وضعیت پرداخت

__4.4 Notification Center__

Schema:

notification

Owner:

Notification

__Table__

__Owner__

__Description__

notifications

Notification

پیام‌های سیستم

notification_templates

Notification

قالب پیام

notification_deliveries

Notification

وضعیت ارسال

notification_queue

Notification

صف ارسال

__5. Layer 2 - Platform Administration Ownership__

Database:

hamareh_saas_core

Schema:

platform

__5.1 Admin Management__

Owner:

Platform Administration

__Table__

__Owner__

admin_users

Platform Administration

admin_roles

Platform Administration

admin_permissions

Platform Administration

admin_user_sessions

Platform Administration

admin_login_attempts

Platform Administration

admin_api_keys

Platform Administration

admin_webhooks

Platform Administration

__5.2 Audit & Logging__

Owner:

Audit

__Table__

__Owner__

audit_logs

Audit

security_events

Audit

data_change_logs

Audit

__5.3 System Configuration__

Owner:

Platform Administration

__Table__

__Owner__

platform_settings

Platform Administration

feature_configurations

Platform Administration

environment_configurations

Platform Administration

__5.4 Support & Ticketing__

Owner:

Support Management

__Table__

__Owner__

support_tickets

Support Management

support_ticket_messages

Support Management

support_ticket_attachments

Support Management

ticket_categories

__Part 2 — Layer 3 Partner \+ Identity Core \+ ERP Foundation__

__6. Layer 3 — Partner / Affiliate Ownership__

Database:

hamareh_saas_core

Schema:

partner

__6.1 Partner Management__

Owner:

Partner Management

__Table__

__Owner__

__Description__

partners

Partner Management

شرکای تجاری پلتفرم

partner_users

Partner Management

کاربران مرتبط با Partner

partner_contacts

Partner Management

اطلاعات تماس Partner

partner_documents

Partner Management

اسناد Partner

partner_bank_accounts

Partner Management

حساب‌های بانکی Partner

partner_activity_logs

Partner Management

تاریخچه فعالیت Partner

__6.2 Partner Commission__

Owner:

Partner Management

__Table__

__Owner__

__Description__

partner_commissions

Partner Management

محاسبه کمیسیون

partner_payouts

Partner Management

پرداخت به Partner

__Layer 3 Rules__

__Rule__

__Decision__

partner_users.user_id

ممنوع

partner_users.tenant_user_id

مجاز

User Entity

مالک Identity Core

Currency Reference

Logical Reference به Master Data

__7. Layer 4 — Identity & Access Core Ownership__

Database:

hamareh_saas_core

Schema:

identity

__7.1 Identity Management__

Owner:

Identity

__Table__

__Owner__

__Description__

users

Identity

هویت اصلی کاربر

credentials

Identity

اطلاعات احراز هویت

profiles

Identity

پروفایل کاربر

user_sessions

Identity

Sessionهای کاربر

__7.2 Tenant Membership__

Owner:

Identity

__Table__

__Owner__

__Description__

tenant_users

Identity

عضویت User در Tenant

__7.3 Authorization__

Owner:

Authorization

__Table__

__Owner__

__Description__

roles

Authorization

نقش‌ها

permissions

Authorization

مجوزها

scopes

Authorization

محدوده دسترسی

role_permissions

Authorization

ارتباط Role و Permission

user_roles

Authorization

تخصیص نقش به User

user_scope_assignments

Authorization

تخصیص Scope

__Identity Ownership Rules__

__Entity__

__Owner__

__وضعیت__

users

Identity

مرجع اصلی

tenant_users

Identity

ارتباط Tenant/User

roles

Authorization

مرجع اصلی

permissions

Authorization

مرجع اصلی

admin_users

Platform Administration

User مستقل مدیریتی

__8. Layer 4 — ERP Core Foundation Ownership__

Database:

hamareh_erp_tenants

__8.1 Organization Management__

Schema:

organization

Owner:

Organization

__Table__

__Owner__

__Description__

erp_companies

Organization

شرکت‌های مشتری

erp_branches

Organization

شعب

erp_departments

Organization

واحدهای سازمانی

organizational_units

Organization

ساختار سازمانی منعطف

organizational_unit_types

Organization

نوع واحد سازمانی

__Organization Rules__

__Entity__

__Decision__

companies

فقط Organization

branches

فقط Organization

departments

فقط Organization

Business Modules

اجازه ایجاد نسخه جدید ندارند

__8.2 Master Data Foundation__

Schema:

master_data

Owner:

Master Data

__Table__

__Owner__

__Description__

master_data_categories

Master Data

دسته‌بندی اطلاعات پایه

master_data_values

Master Data

مقادیر پایه

currencies

Master Data

کاتالوگ ارزها

units_of_measure

Master Data

واحدهای سنجش

countries

Master Data

کشورها

languages

Master Data

زبان‌ها

tax_definitions

Master Data

قوانین مالیاتی

calendars

Master Data

تقویم‌ها

__8.3 Workflow Engine__

Schema:

workflow

Owner:

Workflow

__Table__

__Owner__

workflow_definitions

Workflow

workflow_steps

Workflow

workflow_instances

Workflow

workflow_approvals

Workflow

workflow_history

Workflow

__8.4 Document Management__

Schema:

documents

Owner:

Document Management

__Table__

__Owner__

documents

Document Management

attachments

Document Management

document_versions

Document Management

file_metadata

Document Management

document_sequences

Document Management

__ERP Foundation Rules__

__Entity__

__Owner__

__وضعیت__

users

Identity

Duplicate ممنوع

companies

Organization

Duplicate ممنوع

currencies

Master Data

Duplicate ممنوع

documents

Document Management

Duplicate ممنوع

attachments

Document Management

Framework مرکزی

__Part 3 — Master Data \+ Cross Cutting \+ ERP Business Module Ownership__

__9. Layer 5 — ERP Master Data Ownership__

Database:

hamareh_erp_tenants

Schema:

master_data

__9.1 Business Partner Master Data__

Owner:

Master Data

__Table__

__Owner__

__Description__

business_partners

Master Data

اشخاص و سازمان‌های تجاری

persons

Master Data

اشخاص حقیقی

business_partner_organizations

Master Data

اشخاص حقوقی

entity_addresses

Master Data

آدرس‌ها

entity_contact_points

Master Data

اطلاعات تماس

payment_terms

Master Data

شرایط پرداخت

__Business Partner Rules__

__Entity__

__Decision__

customers

Sales Module Reference

suppliers

Purchase Module Reference

business_partners

Master Data Owner

persons

Master Data Owner

organizations

Master Data Owner

__9.2 Product & Item Master Data__

Owner:

Master Data / Inventory

__Table__

__Owner__

__Description__

items

Inventory

کالا، محصول، ماده اولیه

item_categories

Inventory

دسته‌بندی کالا

item_attributes

Inventory

ویژگی‌های کالا

item_units

Inventory

واحدهای کالا

__Item Ownership Rules__

__Entity__

__Owner__

__وضعیت__

items

Inventory

Source Of Truth

BOM component_item_id

Inventory Reference

Sales Items

Inventory Reference

Purchase Items

Inventory Reference

Manufacturing Items

Inventory Reference

__9.3 Warehouse & Inventory Master Data__

Owner:

Inventory

__Table__

__Owner__

__Description__

warehouses

Inventory

انبارها

warehouse_locations

Inventory

موقعیت‌های داخل انبار

inventory_transactions

Inventory

گردش کالا

inventory_balances

Inventory

موجودی لحظه‌ای

stock_adjustments

Inventory

اصلاح موجودی

__Inventory Ownership Rules__

__Entity__

__Decision__

warehouses

فقط Inventory

warehouse_locations

فقط Inventory

stock

فقط Inventory

Manufacturing

Logical Reference

__9.4 Organization Reference Entities__

Owner:

Organization

__Table__

__Owner__

__Status__

companies

Organization

Existing

branches

Organization

Existing

departments

Organization

Existing

__9.5 HR Master Reference__

Owner:

HR Module

__Table__

__Owner__

__Description__

employees

HR

پرسنل سازمان

employee_profiles

HR

اطلاعات تکمیلی پرسنل

__Employee Ownership Rules__

__Entity__

__Decision__

users

Identity

employees

HR

employee authentication

Identity Reference

employee profile

HR

__9.6 Master Data Extensions__

Owner:

Master Data

__Table__

__Owner__

__Action__

tags

Master Data

ایجاد Framework برچسب

entity_tags

Master Data

ارتباط Entity و Tag

document_sequences

Document Management

شماره‌گذاری اسناد

__10. Cross Cutting Tables__

این جداول توسط کل سیستم استفاده می‌شوند ولی Owner مشخص دارند.

__Table__

__Owner__

__Description__

audit_logs

Audit

ثبت رخدادها

notifications

Notification

پیام‌ها

event_outbox

Event System

انتشار Event

system_logs

Platform Administration

لاگ سیستم

attachments

Document Management

فایل‌های پیوست

__Cross Cutting Rules__

__Rule__

__Decision__

Moduleها اجازه ایجاد Audit Log مستقل ندارند

ممنوع

Moduleها اجازه ایجاد Attachment Table ندارند

ممنوع

Notification Framework مرکزی است

اجباری

Eventها از Event Outbox عبور می‌کنند

اجباری

__11. ERP Business Module Ownership__

Database:

hamareh_erp_tenants

__11.1 Accounting Module__

Schema:

accounting

Owner:

Accounting

__Table__

__Owner__

fin_acc_accounts

Accounting

fin_acc_polymorphic_details

Accounting

fin_acc_cost_centers

Accounting

fin_acc_journal_entries

Accounting

fin_acc_journal_items

Accounting

fin_acc_exchange_rates

Accounting

fin_acc_closing_logs

Accounting

fin_acc_tax_transactions

Accounting

fin_acc_bank_reconciliations

Accounting

fin_acc_budget_headers

Accounting

fin_acc_budget_items

Accounting

fin_acc_fixed_assets

Accounting

fin_acc_depreciation_entries

Accounting

__11.2 Inventory Module__

Schema:

inventory

Owner:

Inventory

__Table__

__Owner__

inventory_movements

Inventory

stock_balances

Inventory

stock_adjustments

Inventory

warehouse_transactions

Inventory

__11.3 Sales Module__

Schema:

sales

Owner:

Sales

__Table__

__Owner__

customers

Sales Reference

leads

Sales

opportunities

Sales

sales_orders

Sales

sales_order_items

Sales

sales_invoices

Sales

sales_invoice_items

Sales

sales_delivery_orders

Sales

return_orders

Sales

__11.4 Procurement Module__

Schema:

purchase

Owner:

Purchase

__Table__

__Owner__

suppliers

Purchase Reference

purchase_requests

Purchase

purchase_request_items

Purchase

purchase_orders

Purchase

purchase_order_items

Purchase

purchase_invoices

Purchase

purchase_invoice_items

Purchase

purchase_receipts

Purchase

__11.5 Manufacturing Module__

Schema:

manufacturing

Owner:

Manufacturing

__Table__

__Owner__

mfg_work_centers

Manufacturing

mfg_boms

Manufacturing

mfg_bom_items

Manufacturing

mfg_bom_versions

Manufacturing

mfg_bom_operations

Manufacturing

mfg_work_orders

Manufacturing

mfg_production_orders

Manufacturing

mfg_production_routing

Manufacturing

mfg_production_logs

Manufacturing

mfg_material_reservations

Manufacturing

mfg_production_costs

Manufacturing

mfg_quality_checks

Manufacturing

__11.6 HR Module__

Schema:

hr

Owner:

HR

__Table__

__Owner__

employees

HR

employee_profiles

HR

attendance_records

HR

payroll_records

HR

hr_documents

HR

__11.7 Project Management Module__

Schema:

projects

Owner:

Project Management

__Table__

__Owner__

projects

Project Management

project_tasks

Project Management

project_members

Project Management

resource_allocations

Project Management

__Part 4 — Entity Governance \+ ERD Rules \+ Final Ownership Statement__

__12. Forbidden Duplicate Entities__

ایجاد Entityهای زیر در Moduleها ممنوع است.

__Entity__

__Owner__

__Rule__

User

Identity

هیچ Module اجازه ایجاد users ندارد

Tenant

Tenant Management

هیچ Module اجازه ایجاد Tenant ندارد

Role

Authorization

هیچ Module اجازه ایجاد Role ندارد

Permission

Authorization

هیچ Module اجازه ایجاد Permission ندارد

Company

Organization

هیچ Module اجازه ایجاد Company ندارد

Branch

Organization

هیچ Module اجازه ایجاد Branch ندارد

Department

Organization

هیچ Module اجازه ایجاد Department ندارد

Currency

Master Data

هیچ Module اجازه ایجاد Currency ندارد

Item

Inventory

هیچ Module اجازه ایجاد Item ندارد

Warehouse

Inventory

هیچ Module اجازه ایجاد Warehouse ندارد

Document

Document Management

هیچ Module اجازه ایجاد Document ندارد

Attachment

Document Management

هیچ Module اجازه ایجاد Attachment ندارد

Employee

HR

هیچ Module اجازه ایجاد Employee ندارد

__13. Missing Entity Resolution__

این بخش برای Entityهایی است که در نسخه‌های اولیه طراحی استفاده شده‌اند ولی مالکیت مشخص نداشتند.

__13.1 Users Resolution__

__Item__

__Decision__

Entity

users

Owner

Identity Core

Database

hamareh_saas_core

Schema

identity

Status

Existing

Action

Duplicate Creation ممنوع

__13.2 Companies Resolution__

__Item__

__Decision__

Entity

erp_companies

Owner

Organization

Database

hamareh_erp_tenants

Schema

organization

Status

Existing

Action

Accounting Reference Only

__13.3 Branches Resolution__

__Item__

__Decision__

Entity

erp_branches

Owner

Organization

Status

Existing

Action

No Duplicate

__13.4 Departments Resolution__

__Item__

__Decision__

Entity

erp_departments

Owner

Organization

Status

Existing

Action

Procurement / HR Reference Only

__13.5 Currencies Resolution__

__Item__

__Decision__

Entity

currencies

Owner

Master Data

Status

Existing

Action

Accounting / Sales / Purchase / Manufacturing Reference

__13.6 Items Resolution__

__Item__

__Decision__

Entity

items

Owner

Inventory

Status

Must Exist

Action

BOM, Sales, Purchase, Manufacturing Reference

__13.7 Warehouses Resolution__

__Item__

__Decision__

Entity

warehouses

Owner

Inventory

Status

Must Exist

Action

Manufacturing and Logistics Reference

__13.8 Employees Resolution__

__Item__

__Decision__

Entity

employees

Owner

HR

Status

Must Exist

Action

Identity Reference Only

__13.9 Attachments Resolution__

__Item__

__Decision__

Entity

attachments

Owner

Document Management

Status

Framework Required

Action

Central Attachment Service

__13.10 Tags Resolution__

__Item__

__Decision__

Entity

tags

Owner

Master Data

Status

Framework Required

Action

Shared Classification System

__14. Foreign Key Policy__

__14.1 Internal Module Relations__

مجاز:

Module A Table

        ↓

Module A Table

مثال:

mfg_bom_items

        FK

mfg_boms

__14.2 Cross Module Relations__

ممنوع:

Sales Table

        FK

Inventory Table

مجاز:

Sales

        ↓

Logical Reference

        ↓

Inventory

مثال:

sales_order_items.item_id

        |

        |

        UUID Reference

        |

        ↓

inventory.items

__15. Logical Reference Policy__

برای Entityهای زیر همیشه Logical Reference استفاده می‌شود:

__Entity__

__Reference Type__

User

user_id

Tenant

tenant_id

Company

company_id

Currency

currency_id

Item

item_id

Warehouse

warehouse_id

Document

document_id

Employee

employee_id

__16. Event Impact Rules__

تمام Moduleهایی که تغییرات مهم ایجاد می‌کنند باید Event منتشر کنند.

__Required Events__

__Domain__

__Event__

Identity

UserCreated

Tenant

TenantCreated

Inventory

ItemUpdated

Inventory

StockChanged

Sales

SalesOrderCreated

Purchase

PurchaseOrderCreated

Accounting

JournalPosted

Manufacturing

ProductionCompleted

HR

EmployeeCreated

__17. Database Ownership Final Map__

SaaS Platform

│

├── Tenant Management

│

├── Subscription

│

├── Billing

│

├── Notification

│

└── Platform Administration

Identity Core

│

├── Users

├── Credentials

├── Profiles

├── Roles

├── Permissions

└── Tenant Membership

ERP Foundation

│

├── Organization

│

├── Master Data

│

├── Workflow

│

└── Document Management

ERP Business Modules

│

├── Accounting

│

├── Inventory

│

├── Sales

│

├── Purchase

│

├── Manufacturing

│

├── HR

│

└── Project Management

__18. ERD Preparation Rules__

قبل از ایجاد ERD برای هر Table موارد زیر باید مشخص باشد:

__Requirement__

__Mandatory__

Database Owner

Yes

Schema

Yes

Module Owner

Yes

Tenant Requirement

Yes

Foreign Key Policy

Yes

Event Impact

Yes

Data Lifecycle

Yes

Soft Delete Policy

Yes

Audit Requirement

Yes

__19. Migration Rules__

تمام Migrationها باید:

- فقط Entityهای Owner شده را ایجاد کنند. 
- Duplicate Entity ایجاد نکنند. 
- Tenant Isolation را رعایت کنند. 
- Naming Convention پروژه را رعایت کنند. 
- Foreign Key بین Moduleها ایجاد نکنند. 
- Event Impact را مشخص کنند. 

__20. Final Ownership Statement__

مدل نهایی مالکیت داده در HamarehERP:

One Entity

        ↓

One Module Owner

        ↓

One Database Authority

        ↓

One Source Of Truth

