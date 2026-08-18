# Database Ownership Matrix v1.0

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Identity & Master Data Layer
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__Database Ownership Matrix__

__Version 1.0__

__Status:__ Architectural Decision Document  
__State:__ Database Ownership Reference  
__Project:__ HamarehERP SaaS Platform

__1. Purpose__

این سند مالکیت رسمی داده‌ها در پلتفرم HamarehERP را مشخص می‌کند.

قوانین:

- هر Table فقط یک Owner دارد. 
- هر Entity فقط در یک Context تعریف می‌شود. 
- هیچ Module اجازه ایجاد نسخه موازی از Entity دیگر ندارد. 
- طراحی ERD باید بر اساس این سند انجام شود. 

__2. Ownership Model__

ساختار مالکیت:

SaaS Platform

        ↓

Identity Core

        ↓

ERP Foundation

        ↓

Business Modules

__3. SaaS Platform Database Ownership__

Database:

hamareh_saas_core

__3.1 Tenant Management Module__

Schema:

saas

__Table__

__Owner__

__Description__

tenants

Tenant Management

مشتری SaaS

tenant_domains

Tenant Management

دامنه مشتری

tenant_settings

Tenant Management

تنظیمات Tenant

tenant_status_history

Tenant Management

تاریخچه وضعیت

مالکیت:

Tenant Management

__3.2 Subscription & Billing Module__

Schema:

billing

__Table__

__Owner__

__Description__

plans

Subscription

پلن‌ها

plan_features

Subscription

امکانات پلن

subscriptions

Subscription

اشتراک مشتری

invoices

Subscription

صورتحساب

billing_transactions

Subscription

تراکنش مالی SaaS

__3.3 Platform Administration__

Schema:

platform

__Table__

__Owner__

platform_settings

Platform Admin

platform_audit_logs

Audit

failed_events

Event System

__4. Identity Core Ownership__

Database:

hamareh_saas_core

Schema:

identity

__4.1 Identity Management__

__Table__

__Owner__

users

Identity

credentials

Identity

profiles

Identity

user_sessions

Identity

__4.2 Authorization__

__Table__

__Owner__

roles

Authorization

permissions

Authorization

scopes

Authorization

role_permissions

Authorization

user_roles

Authorization

user_scopes

Authorization

__5. ERP Foundation Ownership__

Database:

hamareh_erp_tenants

__5.1 Organization Module__

Schema:

organization

__Table__

__Owner__

companies

Organization

branches

Organization

departments

Organization

organization_units

Organization

__5.2 Master Data Module__

Schema:

master_data

مالک:

Master Data

__Table__

__Owner__

currencies

Master Data

exchange_rates

Master Data

units

Master Data

countries

Master Data

languages

Master Data

tax_definitions

Master Data

calendars

Master Data

__5.3 Workflow Module__

Schema:

workflow

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

__5.4 Document Management__

Schema:

documents

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

__6. ERP Business Module Ownership__

Database:

hamareh_erp_tenants

__6.1 Accounting Module__

Schema:

accounting

__Table__

__Owner__

chart_of_accounts

Accounting

accounts

Accounting

fiscal_periods

Accounting

journals

Accounting

journal_entries

Accounting

journal_lines

Accounting

financial_documents

Accounting

__6.2 Inventory Module__

Schema:

inventory

__Table__

__Owner__

items

Inventory

warehouses

Inventory

warehouse_locations

Inventory

stock_balances

Inventory

inventory_movements

Inventory

stock_adjustments

Inventory

__6.3 Sales Module__

Schema:

sales

__Table__

__Owner__

customers

Sales

leads

Sales

opportunities

Sales

sales_orders

Sales

sales_order_items

Sales

sales_history

Sales

__6.4 Purchase Module__

Schema:

purchase

__Table__

__Owner__

suppliers

Purchase

purchase_requests

Purchase

purchase_orders

Purchase

purchase_order_items

Purchase

purchase_receipts

Purchase

__6.5 Manufacturing Module__

Schema:

manufacturing

__Table__

__Owner__

bill_of_materials

Manufacturing

bom_items

Manufacturing

production_orders

Manufacturing

work_orders

Manufacturing

production_transactions

Manufacturing

__6.6 HR Module__

Schema:

hr

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

__6.7 Project Management Module__

Schema:

projects

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

__7. Cross Cutting Tables__

این جداول توسط کل سیستم استفاده می‌شوند اما مالکیت مشخص دارند.

__Table__

__Owner__

audit_logs

Audit

notifications

Notification

event_outbox

Event System

system_logs

Platform Admin

__8. Forbidden Duplicate Entities__

ایجاد موارد زیر در Moduleها ممنوع است:

__Entity__

__Owner__

User

Identity

Tenant

Tenant Management

Role

Authorization

Permission

Authorization

Company

Organization

Currency

Master Data

Document

Document Management

مثال ممنوع:

Inventory Module

\+

 

users table

یا:

Sales Module

\+

roles table

__9. ERD Preparation Rules__

قبل از رسم ERD:

برای هر Table باید مشخص باشد:

1. Database 
2. Schema 
3. Module Owner 
4. Tenant Requirement 
5. Foreign Key Policy 
6. Event Impact 

__10. Final Ownership Statement__

مدل نهایی مالکیت:

One Entity

        ↓

One Module

        ↓

One Database Owner

        ↓

One Source Of Truth

این سند مرجع رسمی طراحی ERD، Schema و Migration خواهد بود.

