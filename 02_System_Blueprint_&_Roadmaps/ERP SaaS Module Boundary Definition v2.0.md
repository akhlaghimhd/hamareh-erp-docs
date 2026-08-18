# ERP SaaS Module Boundary Definition v2.0

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** System Blueprint & Roadmaps
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__ERP SaaS Module Boundary Definition__

__Version 2.0__

__Status:__ Architectural Decision Document  
__State:__ Consolidated Module Ownership Definition  
__Project:__ HamarehERP SaaS Platform

__1. Purpose__

این سند مرزهای رسمی تمام Bounded Contextهای سیستم HamarehERP را مشخص می‌کند.

اهداف:

- تعیین مالکیت Entityها 
- جلوگیری از ایجاد داده‌های تکراری 
- مشخص کردن مسئولیت هر Module 
- تعیین API Ownership 
- تعیین Event Ownership 
- جلوگیری از Coupling مستقیم 

اصل اصلی:

هر قابلیت، هر Entity و هر داده فقط یک مالک دارد.

__2. Global Module Rules__

__Rule 1 - Single Ownership__

هر Entity فقط در یک Module تعریف می‌شود.

مثال:

صحیح:

Invoice

Owner:

Accounting

غلط:

Invoice

Accounting

\+

Sales

__Rule 2 - No Direct Database Access__

هیچ Module اجازه ندارد:

- جدول Module دیگر را Query کند. 
- داده Module دیگر را Update کند. 
- Foreign Key مستقیم ایجاد کند. 

__Rule 3 - Allowed Communication__

ارتباط فقط:

API Contract

یا

Service Contract

یا

Domain Event

یا

Integration Event

__3. SaaS Platform Modules__

__3.1 Tenant Management__

__Responsibility__

مدیریت مشتری SaaS

__Owned Entities__

Tenant

Tenant Profile

Tenant Domain

Tenant Status

Tenant Configuration

__Provides__

API:

Tenant API

Events:

TenantCreated.v1

TenantActivated.v1

TenantSuspended.v1

__Depends On__

None

__3.2 Subscription & Billing__

__Responsibility__

مدیریت مدل درآمد SaaS

__Owned Entities__

Plan

Plan Feature

Subscription

Invoice

Billing Transaction

__Provides__

Subscription API

Billing API

Events:

SubscriptionActivated.v1

SubscriptionExpired.v1

InvoiceCreated.v1

Depends:

Tenant Management

__3.3 Payment Gateway__

__Responsibility__

ارتباط با سیستم‌های پرداخت

Owned:

Payment Transaction

Payment Provider

Provides:

Payment API

Events:

PaymentSucceeded.v1

PaymentFailed.v1

Depends:

Billing

__4. Identity & Security Core__

__4.1 Identity Management__

Owned:

User

Credential

Profile

Session

Provides:

Authentication API

Events:

UserCreated.v1

UserActivated.v1

PasswordChanged.v1

__4.2 Authorization__

Owned:

Role

Permission

Scope

Access Rule

Provides:

Permission Check API

Rules:

تمام Moduleها مصرف‌کننده این بخش هستند.

__5. ERP Foundation Modules__

__5.1 Organization Management__

Owned:

Company

Branch

Department

Organization Tree

Events:

CompanyCreated.v1

BranchCreated.v1

__5.2 Master Data Management__

__Responsibility__

مدیریت داده‌های عمومی مشترک

Owned:

Currency

Unit

Country

Language

Tax Definition

Important:

Master Data مالک تمام داده‌ها نیست.

داده تخصصی Moduleها متعلق به خودشان است.

مثال:

Product:

مالک Inventory

Customer:

مالک Sales

Supplier:

مالک Purchase

__5.3 Workflow Engine__

Owned:

Workflow Definition

Workflow Instance

Approval Step

Approval History

Provides:

Workflow API

Events:

WorkflowCompleted.v1

__5.4 Document Management__

Owned:

Document

Attachment

File Metadata

Provides:

Document API

__6. ERP Business Modules__

__6.1 Accounting__

Owned:

Chart Of Account

Journal

Accounting Entry

Ledger

Financial Document

Payment Allocation

Consumes:

InvoiceCreated

PurchaseCompleted

SalesCompleted

Produces:

JournalCreated.v1

__6.2 Inventory__

Owned:

Warehouse

Stock

Inventory Movement

Stock Adjustment

Inventory Transaction

Consumes:

SalesConfirmed

PurchaseCompleted

Produces:

InventoryChanged.v1

__6.3 Sales__

Owned:

Customer

Lead

Opportunity

Sales Order

Sales Invoice Reference

Produces:

SalesConfirmed.v1

Consumes:

InventoryChanged

Accounting Rules

__6.4 Purchase__

Owned:

Supplier

Purchase Request

Purchase Order

Purchase Receipt

Produces:

PurchaseCompleted.v1

__6.5 Manufacturing__

Owned:

Bill Of Material

Production Order

Work Order

Production Transaction

Consumes:

InventoryChanged

Accounting Rules

Produces:

ProductionCompleted.v1

__6.6 Human Resource__

Owned:

Employee

Attendance

Payroll

HR Document

Depends:

Identity

Organization

__6.7 Project Management__

Owned:

Project

Task

Resource Allocation

Project Cost

Depends:

Identity

Organization

__7. Ownership Matrix__

__Entity Type__

__Owner__

Tenant

Tenant Management

User

Identity

Role

Authorization

Company

Organization

Currency

Master Data

Workflow

Workflow Engine

Document

Document Management

Invoice

Accounting

Stock

Inventory

Sales Order

Sales

Purchase Order

Purchase

BOM

Manufacturing

Employee

HR

Project

Project Management

__8. Final Rule__

هیچ Module جدیدی قبل از مشخص شدن موارد زیر ایجاد نمی‌شود:

1. مسئولیت 
2. مالک Entity 
3. API Contract 
4. Event Contract 
5. Dependency 
6. Tenant Impact 

