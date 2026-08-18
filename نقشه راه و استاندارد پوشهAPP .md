# نقشه راه و استاندارد پوشهAPP

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** General Documentation
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__نقشه راه و استاندارد پوشه‌بندی app (معماری ماژولار مونولیت)__

تمام کدهای برنامه باید در پوشه app/Modules قرار گیرند. هیچ مدل یا کنترلری نباید در روت app سرگردان باشد. علاوه بر این، یک پوشه Base برای کدهای مشترک زیرساختی نیاز داریم.

__۱. نمای کلان پوشه app__

Plaintext

app/

 ├── Base/                           \# ⚙️ کدهای مشترک، زیرساخت و فریم‌ورک

 │    ├── Exceptions/                \# مدیریت یکپارچه خطاها

 │    ├── Traits/                    \# ترِیت‌ها (مثل TenantScoped برای اعمال خودکار RLS)

 │    ├── CoreServices/              \# موتورهای مرکزی (مثل Event Outbox Publisher)

 │    └── Providers/                 \# Service Provider های اصلی سیستم

 │

 └── Modules/                        \# 🏢 ماژول‌های ۹‌گانه (آینه دقیق مایگریشن‌ها)

      │

      ├── SaasPlatform/              \# معادل: saas_platform

      ├── SaasAdmin/                 \# معادل: saas_admin

      ├── PartnerLayer/              \# معادل: partner_layer

      ├── IdentityCore/              \# معادل: identity_core

      ├── MasterData/                \# معادل: master_data

      ├── ProcurementSales/          \# معادل: procurement_sales

      ├── Manufacturing/             \# معادل: manufacturing

      ├── HrManagement/              \# معادل: hr_management

      └── ProjectManagement/         \# معادل: project_management

__۲. ساختار داخلی استاندارد برای هر ماژول (بسیار مهم)__

برای اینکه "قانون دور نزدن API" و "تفکیک منطق" رعایت شود، __داخل هر یک از ۹ ماژول بالا__ باید دقیقاً پوشه‌های زیر ایجاد شوند (مثال برای ماژول ProcurementSales):

Plaintext

app/Modules/ProcurementSales/

 ├── Controllers/          \# فقط دریافت Request، فراخوانی کلاس ولیدیشن و پاس دادن به Service

 ├── Services/             \# قلب تپنده ماژول\! (کدهای محاسبه قیمت، تایید فاکتور، کسر موجودی)

 ├── Models/               \# مدل‌های Eloquent (مثل PurchaseOrder, SalesOrder)

 ├── DTOs/                 \# (Data Transfer Objects) فرمت‌بندی امن داده‌ها بین Controller و Service

 ├── Events/               \# رویدادهایی که این ماژول شلیک می‌کند (مثل SalesOrderCreated)

 ├── Listeners/            \# شنونده‌ها برای واکنش به رویدادهای سایر ماژول‌ها (بدون کوپلینگ مستقیم)

 ├── Requests/             \# کلاس‌های FormRequest لاراول برای اعتبارسنجی ورودی API

 └── Routes/               \# فایل api.php اختصاصی همین ماژول

__🛡️ ۵ قانون طلایی اجرایی در این ساختار (Red Lines)__

بر اساس اسناد معماری که آپلود کردید، تیم توسعه در هنگام کدنویسی داخل این پوشه‌ها باید قوانین زیر را به صورت اجباری رعایت کند:

1. __قانون TenantScoped (ایزوله‌سازی مستأجر):__ تمامی مدل‌های داخل ماژول‌های ۳ تا ۹ (مثل مدل Employee در HrManagement یا MfgWorkCenter در Manufacturing) باید از یک Trait به نام TenantScoped استفاده کنند. این Trait به صورت خودکار where('tenant_id', current_tenant) را به تمامی کوئری‌ها اضافه می‌کند تا نشت داده (Data Bleed) رخ ندهد.
2. __قانون ارجاع منطقی (بدون FK فیزیکی بین ماژول‌ها):__ هیچ مدلی در پوشه ProcurementSales حق ندارد belongsTo مستقیم به مدلی در پوشه IdentityCore یا MasterData بزند که باعث ایجاد کلید خارجی فیزیکی شود. ارتباطات بین ماژولی صرفاً از طریق ذخیره UUID (مثلاً business_partner_id) و واکشی داده‌ها از طریق کلاس‌های Service ماژولِ مقصد انجام می‌شود.
3. __قانون کنترلر لاغر، سرویس چاق (API First):__ هیچ‌گونه دستور If/Else بیزینسی، کوئری مستقیم دیتابیس (Model::where...) یا محاسبه‌ای نباید داخل پوشه Controllers نوشته شود. کنترلر فقط یک نامه رسان است.
4. __قانون Single Source of Truth برای Master Data:__ مشتریان، پیمانکاران، کالاها و ساختار سازمانی فقط در ماژول MasterData ساخته و مدیریت می‌شوند. ماژول ProjectManagement یا HrManagement حق ندارند جدول یا مدل موازی برای نگهداری اطلاعات شرکت‌ها یا افراد بسازند.
5. __ارتباط ناهمگام (Event-Driven):__ اگر قرار است با ثبت یک دستور تولید (در Manufacturing)، موجودی متریال چک شود، ماژول تولید نباید مستقیماً متدهای ماژول انبار را صدا بزند؛ بلکه یک رویداد (Event) منتشر می‌کند، و ماژول مربوطه از طریق پوشه Listeners خود آن را می‌شنود و اقدام می‌کند.

