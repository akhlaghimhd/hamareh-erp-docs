# ERP SaaS Database Design Roadmap Phase 2

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** System Blueprint & Roadmaps
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__ERP SaaS Database Design Roadmap Phase 2__

__Version 2.0__

__Status:__ Architectural Decision Document  
__State:__ Database Planning Reference  
__Project:__ HamarehERP SaaS Platform

__1. Purpose__

این سند مسیر رسمی طراحی پایگاه داده HamarehERP را مشخص می‌کند.

هدف:

- تعیین مرزهای دیتابیس 
- تعیین مالکیت جداول 
- جلوگیری از طراحی اشتباه Schema 
- هماهنگ‌سازی Database با معماری Bounded Context 
- آماده‌سازی برای Migration در Laravel 
- حفظ قابلیت توسعه آینده 

اصل پایه:

Database باید بازتاب معماری سیستم باشد، نه محدودیت طراحی آن.

__2. Database Architecture Model__

معماری دیتابیس از دو بخش اصلی تشکیل می‌شود:

Database A

hamareh_saas_core

مالک:

Layer 1

SaaS Platform

Layer 2

Identity & Security Core

================================

Database B

hamareh_erp_tenants

مالک:

Layer 3

ERP Foundation

Layer 4

ERP Business Modules

__3. Database Ownership Rules__

__Rule 1 - Single Table Ownership__

هر جدول فقط یک Owner دارد.

مثال صحیح:

sales_orders

Owner:

Sales Module

مثال غلط:

sales_orders

Owner:

Sales \+ Inventory

__Rule 2 - Logical Database Separation__

حتی اگر تمام جدول‌ها در یک PostgreSQL قرار داشته باشند:

هر Module باید Schema منطقی مستقل داشته باشد.

نمونه:

hamareh_erp_tenants

├── organization

├── master_data

├── accounting

├── inventory

├── sales

├── purchase

├── manufacturing

└── hr

__Rule 3 - Physical Foreign Key Policy__

__داخل یک Module__

Physical FK مجاز است.

مثال:

accounting.journal

        |

        FK

accounting.journal_items

__بین Moduleها__

Physical FK ممنوع است.

مثال ممنوع:

sales.orders

FK

inventory.stock

ارتباط:

- UUID Reference 
- API 
- Event 

__4. Database Layer Mapping__

__Layer 1 - SaaS Platform Database__

Database:

hamareh_saas_core

Schema:

saas

Tables:

__Tenant Management__

مالک:

tenants

tenant_domains

tenant_settings

tenant_status_history

__Subscription__

مالک:

plans

plan_features

subscriptions

subscription_history

invoices

billing_transactions

__Platform Administration__

مالک:

platform_users

system_settings

platform_audit_logs

__5. Layer 2 - Identity Core Database__

Database:

hamareh_saas_core

Schema:

identity

Tables:

users

credentials

profiles

roles

permissions

scopes

user_roles

user_permissions

tenant_memberships

__6. Layer 3 - ERP Foundation Database__

Database:

hamareh_erp_tenants

Schema:

foundation

__Organization Module__

Tables:

companies

branches

departments

organization_units

تمام جداول:

اجباری:

tenant_id

__Master Data Module__

Schema:

master_data

Tables:

currencies

units

countries

languages

tax_definitions

measurement_units

__Workflow Module__

Schema:

workflow

Tables:

workflow_definitions

workflow_steps

workflow_instances

workflow_approvals

workflow_history

__Document Module__

Schema:

documents

Tables:

documents

attachments

file_metadata

document_versions

__7. Layer 4 - Business Database__

Database:

hamareh_erp_tenants

هر Module دارای Schema مستقل:

accounting

inventory

sales

purchase

manufacturing

hr

projects

__8. Accounting Database Boundary__

Schema:

accounting

مالک:

chart_of_accounts

accounts

journals

journal_entries

journal_lines

financial_periods

cost_centers

ارتباط با:

Sales

Purchase

Inventory

فقط از طریق:

- API 
- Event 

__9. Inventory Database Boundary__

Schema:

inventory

مالک:

warehouses

locations

items

stock_balances

inventory_movements

stock_adjustments

__10. Sales Database Boundary__

Schema:

sales

مالک:

customers

leads

opportunities

sales_orders

sales_order_items

sales_status_history

__11. Purchase Database Boundary__

Schema:

purchase

مالک:

suppliers

purchase_requests

purchase_orders

purchase_order_items

purchase_receipts

__12. Manufacturing Database Boundary__

Schema:

manufacturing

مالک:

bill_of_materials

production_orders

work_orders

production_transactions

__13. HR Database Boundary__

Schema:

hr

مالک:

employees

attendance

payroll

hr_documents

__14. Tenant Isolation Database Rules__

تمام جدول‌های Layer 3 و Layer 4:

الزامی:

tenant_id UUID NOT NULL

همراه با:

PostgreSQL Row Level Security

\+

Application Tenant Context

__15. Standard Audit Columns__

تمام جدول‌های عملیاتی:

حداقل:

id UUID

tenant_id UUID

created_at

updated_at

created_by

updated_by

در جداول حساس:

deleted_at

version

__16. Primary Key Strategy__

تمام Entityهای Business:

استفاده از:

UUID Primary Key

دلایل:

- SaaS Multi Tenant 
- Distributed Future 
- Microservice Migration 

__17. Database Design Sequence__

__Phase 1__

Foundation Database

ایجاد:

1. Tenant Context 
2. Organization 
3. Identity Integration 
4. Master Data 
5. Audit Structure 

__Phase 2__

First Business Module

Accounting:

- Chart Of Account 
- Journal 
- Ledger 

__Phase 3__

Inventory:

- Warehouse 
- Stock 
- Movement 

__Phase 4__

Operational Modules:

- Sales 
- Purchase 
- Manufacturing 

__18. Final Database Principle__

معماری نهایی:

Architecture

        ↓

Bounded Context

        ↓

Database Ownership

        ↓

Schema Design

        ↓

Tables

        ↓

Migration

هیچ جدول جدیدی بدون مشخص شدن:

- Owner 
- Context 
- Dependency 
- Tenant Impact 

ایجاد نخواهد شد.

