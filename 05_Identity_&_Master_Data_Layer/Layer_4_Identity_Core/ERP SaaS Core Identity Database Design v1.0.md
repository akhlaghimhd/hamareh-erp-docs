# ERP SaaS Core Identity Database Design v1.0

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Identity & Master Data Layer
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__ERP SaaS Core Identity Database Design__

__Version 1.0__

__Status:__ Architectural Decision Document  
__State:__ Database Design Reference  
__Project:__ HamarehERP SaaS Platform

__1. Purpose__

این سند طراحی دیتابیس هسته هویت و دسترسی HamarehERP را مشخص می‌کند.

این لایه مسئول:

- مدیریت کاربران 
- احراز هویت 
- عضویت کاربران در Tenantها 
- نقش‌ها 
- مجوزها 
- Scopeهای دسترسی 
- کنترل محدوده سازمانی 

است.

این لایه توسط تمام Moduleهای ERP مصرف می‌شود و هیچ Module اجازه ایجاد سیستم Identity مستقل ندارد.

__2. Identity Architecture Model__

مدل دسترسی:

User

 ↓

Tenant Membership

 ↓

Role Assignment

 ↓

Permission

 ↓

Scope

 ↓

Resource

__3. Database Location__

Database:

hamareh_saas_core

Schema:

identity

__4. Core Identity Tables__

__4.1 users__

مالک:

Identity Management

هدف:

نگهداری موجودیت اصلی کاربر سیستم.

Structure:

__Column__

__Type__

__Description__

id

UUID

Primary Key

username

varchar

شناسه کاربر

email

varchar

ایمیل

phone

varchar

شماره تماس

status

enum

وضعیت کاربر

created_at

timestamp

زمان ایجاد

updated_at

timestamp

آخرین تغییر

__4.2 credentials__

مالک:

Identity Management

هدف:

نگهداری اطلاعات احراز هویت.

Structure:

__Column__

__Type__

id

UUID

user_id

UUID

password_hash

varchar

last_password_change

timestamp

failed_attempts

integer

locked_until

timestamp

قانون:

Password خام هرگز ذخیره نمی‌شود.

__4.3 user_profiles__

مالک:

Identity Management

هدف:

اطلاعات تکمیلی کاربر.

Structure:

__Column__

__Type__

id

UUID

user_id

UUID

first_name

varchar

last_name

varchar

avatar

varchar

language

varchar

__5. Tenant Membership Model__

__5.1 tenant_memberships__

مالک:

Identity Core

هدف:

اتصال User به Tenant.

Structure:

__Column__

__Type__

id

UUID

tenant_id

UUID

user_id

UUID

status

enum

joined_at

timestamp

قانون:

یک User می‌تواند عضو چند Tenant باشد.

مثال:

User A

 ├── Tenant 1

 |

 └── Tenant 2

__6. Role Management__

__6.1 roles__

مالک:

Authorization

هدف:

تعریف نقش‌های سیستم.

Structure:

__Column__

__Type__

id

UUID

name

varchar

scope_type

varchar

description

text

نمونه:

Admin

Accountant

Warehouse Manager

Sales Manager

__6.2 permissions__

مالک:

Authorization

هدف:

تعریف مجوزهای عملیاتی.

Structure:

__Column__

__Type__

id

UUID

code

varchar

module

varchar

action

varchar

نمونه:

inventory.stock.view

inventory.stock.adjust

sales.order.create

__6.3 role_permissions__

مالک:

Authorization

ارتباط:

Role

  ↓

Permission

Structure:

__Column__

__Type__

role_id

UUID

permission_id

UUID

__7. User Role Assignment__

__7.1 user_roles__

هدف:

اختصاص Role به User.

Structure:

__Column__

__Type__

id

UUID

user_id

UUID

tenant_id

UUID

role_id

UUID

مثال:

Ali

Tenant A

Role:

Warehouse Manager

__8. Scope Driven Access Model__

Scope قلب امنیت داده در ERP است.

مدل:

User

 ↓

Role

 ↓

Scope

 ↓

Allowed Resource

__9. Scope Tables__

__9.1 scopes__

مالک:

Authorization

Structure:

__Column__

__Type__

id

UUID

name

varchar

type

varchar

Scope Type:

Company

Branch

Department

Warehouse

Project

__9.2 user_scopes__

هدف:

محدود کردن دسترسی User.

Structure:

__Column__

__Type__

user_id

UUID

scope_id

UUID

tenant_id

UUID

مثال:

کاربر:

Ali

Scope:

Warehouse A

نتیجه:

کاربر فقط Warehouse A را می‌بیند.

__10. Organization Access Model__

ارتباط:

Tenant

 ↓

Company

 ↓

Branch

 ↓

Department

 ↓

Warehouse

Identity فقط Reference نگهداری می‌کند.

مالک Organization:

ERP Foundation

__11. JWT Context Model__

هر Request باید شامل Context امنیتی باشد:

{

 "user_id": "uuid",

 "tenant_id": "uuid",

 "roles": [],

 "scopes": []

}

__12. Tenant Security Rules__

تمام Queryهای ERP باید:

1. Tenant Context دریافت کنند. 
2. tenant_id را اعمال کنند. 
3. PostgreSQL RLS فعال باشد. 

بدون Tenant Context:

Request رد می‌شود.

__13. Permission Evaluation Flow__

فرآیند:

Request

 ↓

Authenticate User

 ↓

Load Tenant Membership

 ↓

Load Roles

 ↓

Resolve Permissions

 ↓

Validate Scope

 ↓

Execute Action

__14. Audit Integration__

تمام عملیات حساس Identity باید Event تولید کند.

Events:

UserCreated.v1

UserLoginSucceeded.v1

UserLoginFailed.v1

PasswordChanged.v1

RoleAssigned.v1

PermissionChanged.v1

__15. Database Constraints__

__کاربران__

Email:

Unique

__Membership__

ترکیب:

tenant_id \+ user_id

Unique است.

__Roles__

Role Name:

در Scope Tenant باید Unique باشد.

__16. Forbidden Design__

موارد ممنوع:

__Module User Table__

مثال:

sales.users

ممنوع.

__Module Permission Table__

مثال:

inventory_permissions

ممنوع.

__Module Authentication__

هر Module باید از Identity Core استفاده کند.

__17. Final Identity Architecture__

مدل نهایی:

Identity Core

├── Users

├── Credentials

├── Profiles

├── Tenant Membership

├── Roles

├── Permissions

├── Scopes

└── Access Rules

