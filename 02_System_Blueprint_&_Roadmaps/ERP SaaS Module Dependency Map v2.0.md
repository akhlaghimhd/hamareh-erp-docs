# ERP SaaS Module Dependency Map v2.0

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** System Blueprint & Roadmaps
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__ERP SaaS Module Dependency Map__

__Version 2.0__

__Status:__ Architectural Decision Document  
__State:__ Consolidated Dependency Definition  
__Project:__ HamarehERP SaaS Platform

__1. Purpose__

این سند روابط وابستگی بین Moduleهای HamarehERP را مشخص می‌کند.

اهداف:

- جلوگیری از Dependency اشتباه 
- جلوگیری از Circular Dependency 
- تعیین ترتیب توسعه 
- مشخص کردن جهت ارتباطات 
- حفظ قابلیت مهاجرت آینده به Microservices 

اصل پایه:

Dependency فقط از بالا به پایین است.

__2. Dependency Direction Model__

ساختار نهایی:

SaaS Platform Layer

        ↓

Identity & Security Core

        ↓

ERP Foundation Layer

        ↓

ERP Business Modules

        ↓

Extensions & Integrations

__3. Dependency Rules__

__Rule 1__

Layer پایین‌تر اجازه Dependency به Layer بالاتر ندارد.

مثال غیرمجاز:

Identity

   ↓

Accounting

Identity نباید چیزی درباره Accounting بداند.

__Rule 2__

Core مستقل است.

Identity نباید وابسته باشد به:

- Inventory 
- Sales 
- Accounting 
- Manufacturing 

__Rule 3__

Business Moduleها فقط از Core و Foundation استفاده می‌کنند.

مثال:

مجاز:

Inventory

 ↓

Organization

 ↓

Identity

غیرمجاز:

Inventory

 ↓

Sales

 ↓

Purchase

__4. Global Dependency Graph__

                 SaaS Platform

        Tenant Management

                |

                |

      Subscription & Billing

                |

                |

        Payment Gateway

                     ↓

             Identity Core

       Identity Management

       Authorization

       Scope Management

                     ↓

           ERP Foundation

       Organization

       Master Data

       Workflow

       Document Management

                     ↓

          ERP Business Modules

       Accounting

       Inventory

       Sales

       Purchase

       Manufacturing

       HR

       Project Management

__5. SaaS Platform Dependencies__

__Tenant Management__

Dependency:

None

Used By:

Identity

Subscription

Billing

Notification

ERP Foundation

Communication:

- API 
- Logical Reference 

__Subscription & Billing__

Depends On:

Tenant Management

Used By:

All Licensed Modules

Responsibility:

- Subscription State 
- Feature Availability 
- License Check 

__Payment Gateway__

Depends On:

Subscription & Billing

Communication:

InvoiceCreated.v1

PaymentSucceeded.v1

__6. Identity Core Dependencies__

__Identity Management__

Depends On:

Tenant Management

Provides:

User Context

Authentication

JWT

Session

Used By:

All Modules

__Authorization__

Depends On:

Identity Management

Provides:

Role

Permission

Scope Validation

Used By:

All Modules

__7. ERP Foundation Dependencies__

__Organization Management__

Depends On:

Tenant Management

Identity Core

Used By:

All ERP Modules

Provides:

Company

Branch

Department Context

__Master Data__

Depends On:

Organization

Used By:

Accounting

Inventory

Sales

Purchase

Manufacturing

__Workflow Engine__

Depends On:

Identity

Organization

Used By:

All Modules requiring Approval

__Document Management__

Depends On:

Tenant

Identity

Used By:

All Modules

__8. Business Module Dependencies__

__Accounting__

Depends On:

Identity

Organization

Master Data

Document Management

Consumed By:

Sales

Purchase

Inventory

Manufacturing

Produces:

JournalCreated.v1

__Inventory__

Depends On:

Identity

Organization

Master Data

Document Management

Consumed By:

Sales

Purchase

Manufacturing

Produces:

InventoryChanged.v1

__Sales__

Depends On:

Identity

Organization

Master Data

Inventory

Accounting

Produces:

SalesConfirmed.v1

__Purchase__

Depends On:

Identity

Organization

Master Data

Inventory

Accounting

Produces:

PurchaseCompleted.v1

__Manufacturing__

Depends On:

Identity

Organization

Inventory

Accounting

Produces:

ProductionCompleted.v1

__HR__

Depends On:

Identity

Organization

No dependency on:

Accounting

Inventory

Sales

__Project Management__

Depends On:

Identity

Organization

No dependency on Business Modules.

__9. Forbidden Dependencies__

موارد زیر ممنوع هستند:

__Business To Business Direct Dependency__

غیرمجاز:

Sales

 ↓

Inventory Database

__Database Coupling__

غیرمجاز:

Sales.orders

FK

Inventory.stock

__Circular Dependency__

غیرمجاز:

Sales

 ↓

Accounting

 ↓

Sales

__10. Communication Strategy__

__Query Operations__

استفاده از:

API Contract

Service Interface

مثال:

Sales:

"آیا مشتری اعتبار دارد؟"

درخواست:

Accounting Credit Service

__State Changes__

استفاده از:

Domain Event

مثال:

SalesConfirmed.v1

        ↓

Inventory

        ↓

Accounting

__11. Development Order__

__Phase 0__

Infrastructure

شامل:

- Repository 
- Docker 
- CI/CD 
- Base Application 

__Phase 1__

SaaS Core

ترتیب:

1. Tenant Management 
2. Identity 
3. Authorization 
4. Subscription 
5. Billing 
6. Notification 
7. Audit 

__Phase 2__

ERP Foundation

ترتیب:

1. Organization 
2. Master Data 
3. Document Management 
4. Workflow 

__Phase 3__

Business Modules

ترتیب:

__1. Accounting__

دلیل:

مرجع مالی تمام عملیات.

__2. Inventory__

دلیل:

وابستگی عملیاتی بالا.

__3. Sales__

__4. Purchase__

__5. Manufacturing__

__6. HR__

__7. Project Management__

__12. Future Evolution Path__

معماری فعلی:

Modular Monolith

مسیر آینده:

Modular Monolith

        ↓

Service Oriented Architecture

        ↓

Microservices

بدون بازطراحی کامل.

__13. Final Dependency Statement__

قانون نهایی:

Platform

    ↓

Core

    ↓

Foundation

    ↓

Business Modules

    ↓

Extensions

هیچ توسعه‌ای خارج از این مسیر مجاز نیست.

