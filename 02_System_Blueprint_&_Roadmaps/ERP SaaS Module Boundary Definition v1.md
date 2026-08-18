# ERP SaaS Module Boundary Definition v1

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** System Blueprint & Roadmaps
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__ERP SaaS Module Boundary Definition v1.0__

__وضعیت سند__

__عنوان:__ سند مرزبندی ماژول‌های ERP SaaS  
__نسخه:__ 1.0 Draft  
__هدف:__ تعیین مسئولیت، مالکیت داده، مرزهای عملکردی و قوانین ارتباطی هر ماژول

__1. هدف سند__

این سند مشخص می‌کند:

- هر ماژول دقیقاً مسئول چه بخشی از سیستم است.
- مالک هر داده (Data Ownership) کدام ماژول است.
- هر Entity فقط در یک ماژول تعریف می‌شود.
- هر API متعلق به کدام ماژول است.
- هر Event توسط کدام ماژول تولید یا مصرف می‌شود.
- هیچ دو ماژولی مسئول یک قابلیت مشترک نخواهند بود.

__2. قوانین عمومی__

__اصل 1__

هر Entity فقط یک مالک دارد.

__اصل 2__

هیچ Entity نباید در دو ماژول ایجاد یا ویرایش شود.

__اصل 3__

تمام عملیات Create / Update / Delete فقط توسط ماژول مالک انجام می‌شود.

__اصل 4__

سایر ماژول‌ها فقط از API یا Event استفاده می‌کنند.

__اصل 5__

دسترسی مستقیم به جداول سایر ماژول‌ها ممنوع است.

__اصل 6__

هر ماژول پایگاه داده منطقی مستقل دارد، حتی اگر همه جداول فعلاً در یک PostgreSQL قرار داشته باشند.

__3. ساختار استاندارد هر ماژول__

هر ماژول باید شامل بخش‌های زیر باشد:

- Domain
- Application
- Infrastructure
- API
- Event Publisher
- Event Consumer
- Repository
- Validation
- Authorization
- License Check
- Configuration

__4. Tenant Management__

__مسئولیت__

مدیریت مشتریان سامانه SaaS

__مالک داده‌ها__

- Tenant
- Tenant Domain
- Tenant Wallet

__مسئول عملیات__

- ایجاد Tenant
- فعال/غیرفعال کردن Tenant
- مدیریت دامنه
- وضعیت سرویس

__APIهای متعلق به ماژول__

- Tenant Management API
- Domain API
- Tenant Status API

__Eventهای تولیدی__

- TenantCreated
- TenantActivated
- TenantSuspended
- TenantDeleted

__Eventهای مصرفی__

ندارد

__5. Identity & Access__

__مسئولیت__

مدیریت هویت و کنترل دسترسی

__مالک داده‌ها__

- User
- Credential
- Profile
- Role
- Permission
- Scope
- User Role
- User Scope
- Tenant User

__مسئول عملیات__

- Login
- Logout
- Password
- MFA
- JWT
- Permission Check

__API__

- Authentication
- Authorization
- User Profile
- Permission Check

__Eventهای تولیدی__

- UserCreated
- UserActivated
- UserLocked
- PasswordChanged
- UserLoggedIn

__Eventهای مصرفی__

- TenantCreated

__6. Subscription & Billing__

__مسئولیت__

مدیریت اشتراک

__مالک داده‌ها__

- Plan
- Subscription
- Invoice
- Coupon
- Wallet Transaction

__مسئول عملیات__

- خرید پلن
- تمدید
- لغو
- صورتحساب

__API__

- Subscription API
- Billing API

__Eventهای تولیدی__

- SubscriptionActivated
- SubscriptionExpired
- InvoiceCreated
- InvoicePaid

__Eventهای مصرفی__

- PaymentSucceeded

__7. Payment Gateway__

__مسئولیت__

ارتباط با درگاه‌های پرداخت

__مالک داده‌ها__

- Payment Transaction

__API__

- Payment Request
- Payment Verify

__Eventهای تولیدی__

- PaymentSucceeded
- PaymentFailed

__Eventهای مصرفی__

- InvoiceCreated

__8. Notification Center__

__مسئولیت__

ارسال اعلان‌ها

__مالک داده‌ها__

- Notification
- Notification Template
- Delivery Queue

__API__

- Notification API

__Eventهای تولیدی__

- NotificationSent

__Eventهای مصرفی__

تمام Eventهای مهم سیستم

__9. Audit__

__مسئولیت__

ثبت رویدادهای سیستمی

__مالک داده‌ها__

- Audit Log

__API__

فقط Query

__Eventهای مصرفی__

تمام Eventهای سیستم

__10. ERP Organization__

__مسئولیت__

ساختار سازمانی مشتری

__مالک داده‌ها__

- Company
- Branch
- Department

__API__

Organization API

__Eventهای تولیدی__

- CompanyCreated
- BranchCreated
- DepartmentCreated

__Eventهای مصرفی__

- TenantCreated

__11. Master Data__

__مسئولیت__

نگهداری اطلاعات پایه

__مالک داده‌ها__

تمام Master Dataهای ERP

__API__

Master Data API

__Eventهای تولیدی__

MasterDataChanged

__12. Workflow__

__مسئولیت__

فرآیندهای تأیید

__مالک داده‌ها__

- Workflow
- Workflow Instance
- Approval

__API__

Workflow API

__Eventهای تولیدی__

WorkflowCompleted

__13. Document Management__

__مسئولیت__

مدیریت اسناد

__مالک داده‌ها__

- Document
- Attachment

__API__

Document API

__Eventهای تولیدی__

DocumentCreated

__14. Accounting__

__مسئولیت__

ثبت عملیات مالی

__مالک داده‌ها__

تمام Entityهای حسابداری

__API__

Accounting API

__Eventهای تولیدی__

JournalCreated

__Eventهای مصرفی__

- InvoicePaid
- SalesCompleted
- PurchaseCompleted

__15. Inventory__

__مسئولیت__

مدیریت موجودی

__مالک داده‌ها__

تمام Entityهای انبار

__API__

Inventory API

__Eventهای تولیدی__

InventoryChanged

__Eventهای مصرفی__

SalesConfirmed

__16. Sales__

__مسئولیت__

مدیریت فروش

__مالک داده‌ها__

تمام Entityهای فروش

__API__

Sales API

__Eventهای تولیدی__

SalesConfirmed

__17. Purchase__

__مسئولیت__

مدیریت خرید

__مالک داده‌ها__

تمام Entityهای خرید

__API__

Purchase API

__Eventهای تولیدی__

PurchaseCompleted

__18. HR__

__مسئولیت__

مدیریت منابع انسانی

__مالک داده‌ها__

تمام Entityهای منابع انسانی

__API__

HR API

__19. Manufacturing__

__مسئولیت__

مدیریت تولید

__مالک داده‌ها__

تمام Entityهای تولید

__API__

Manufacturing API

__20. Project Management__

__مسئولیت__

مدیریت پروژه

__مالک داده‌ها__

تمام Entityهای پروژه

__API__

Project API

__21. قوانین ارتباط بین ماژول‌ها__

تمام ارتباطات باید از یکی از روش‌های زیر انجام شود:

- API
- Domain Event
- Integration Event

هیچ ارتباط دیگری مجاز نیست.

__22. قوانین مالکیت داده__

- هر جدول فقط متعلق به یک ماژول است.
- هر API فقط متعلق به یک ماژول است.
- هر Event فقط یک Publisher دارد.
- هر Entity فقط یک مالک دارد.
- هیچ ماژولی اجازه تغییر مستقیم داده‌های ماژول دیگر را ندارد.

__23. نتیجه نهایی__

این سند مرز دقیق تمامی ماژول‌های سیستم را مشخص می‌کند و از این مرحله به بعد، تمام طراحی‌های دیتابیس، API، Backend و Frontend باید بر اساس این مرزبندی انجام شوند.

هر قابلیت جدید، پیش از طراحی، ابتدا باید در این سند جایگاه مشخص خود را پیدا کند.

