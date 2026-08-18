# Business Partner Architecture Standard v1.0

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** Identity & Master Data Layer
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__ERP.Core Master Data__

__Business Partner Architecture v1.0__

__Status: Locked  
Document Type: Architecture Standard__

__==================================================__

__1. هدف__

__==================================================__

__Business Partner یکی از موجودیت‌های پایه ERP.Core Master Data است و مسئول نگهداری اطلاعات طرف‌های تجاری یک Tenant می‌باشد.__

__این موجودیت باید بتواند سناریوهای مختلف ERP شامل:__

- __Customer__
- __Supplier__
- __Contractor__
- __Service Provider__
- __Carrier__
- __سایر طرف‌های تجاری__

__را پشتیبانی کند.__

__==================================================__

__2. مالکیت داده__

__==================================================__

__در معماری Multi-Tenant:__

__هر Tenant مالک کامل داده‌های Business Partner خود است.__

__مدل:__

__Tenant__

__|__

__\+-- Business Partners__

__اطلاعات Business Partner بین Tenantها به اشتراک گذاشته نمی‌شود.__

__دلایل:__

- __حفظ محرمانگی اطلاعات__
- __جلوگیری از افشای روابط تجاری__
- __رعایت Tenant Isolation__

__==================================================__

__3. Tenant Ownership and Global Directory Policy__

__==================================================__

__در نسخه فعلی معماری:__

__Global Business Directory وجود ندارد.__

__اگر یک شرکت در چند Tenant مختلف ثبت شود، هر Tenant رکورد مستقل خود را خواهد داشت.__

__مثال:__

__Tenant A:__

__شرکت پارس__

__Role:  
Supplier__

__Tenant B:__

__شرکت پارس__

__Role:  
Customer__

__این دو رکورد مستقل هستند.__

__==================================================__

__4. مفهوم اصلی Business Partner__

__==================================================__

__Business Partner یک موجودیت عمومی است و نباید فقط به یک نقش محدود شود.__

__یک Business Partner می‌تواند چند Role داشته باشد.__

__مثال:__

__شرکت پارس:__

__Roles:__

- __Supplier__
- __Contractor__
- __Customer__

__==================================================__

__5. انواع Party__

__==================================================__

__Business Partner می‌تواند شامل:__

__Person__

__مثال:__

- __مشاور مستقل__
- __پیمانکار حقیقی__

__Organization__

__مثال:__

- __شرکت تولیدی__
- __تامین‌کننده__
- __مشتری سازمانی__

__==================================================__

__6. تفکیک Role و Profile__

__==================================================__

__Role فقط نوع رابطه Business Partner با Tenant را مشخص می‌کند.__

__مثال:__

__Business Partner Role:__

- __Customer__
- __Supplier__
- __Contractor__
- __Carrier__
- __Service Provider__

__اطلاعات اختصاصی هر نقش در Profile مربوط به آن Role نگهداری می‌شود.__

__مثال:__

__Customer Profile:__

- __Credit Limit__
- __Payment Terms__

__Supplier Profile:__

- __Purchase Terms__
- __Lead Time__

__Contractor Profile:__

- __Service Information__

__مالکیت Profileهای تخصصی توسط ماژول مربوطه مدیریت می‌شود.__

__==================================================__

__7. Contacts__

__==================================================__

__Business Partner می‌تواند چند Contact داشته باشد.__

__Contact یک موجودیت مستقل است.__

__مثال:__

__شرکت چاپ شرق:__

__Contacts:__

- __مدیر تولید__
- __مسئول مالی__
- __مسئول خرید__

__Contact Person الزاماً User سیستم نیست.__

__در آینده امکان اتصال:__

__Contact Person__

__|__

__External Access Mapping__

__|__

__Identity Layer__

__|__

__External User Account__

__وجود خواهد داشت.__

__==================================================__

__8. Addresses__

__==================================================__

__Business Partner می‌تواند چند Address داشته باشد.__

__مثال:__

- __دفتر مرکزی__
- __کارخانه__
- __انبار__
- __آدرس مالی__

__Location به عنوان Business Partner جدا ایجاد نمی‌شود.__

__==================================================__

__9. Relationships__

__==================================================__

__Business Partner می‌تواند Relationship داشته باشد.__

__نمونه‌ها:__

- __Parent / Subsidiary__
- __Related Company__
- __Branch Relationship__

__Relationshipهای تخصصی ماژول‌ها مانند:__

- __قرارداد پیمانکاری__
- __حمل‌ونقل__
- __همکاری تولیدی__

__در ماژول مربوطه مدیریت می‌شوند.__

__Relationshipهای Business Partner نیز Tenant-Owned هستند و بین Tenantها به اشتراک گذاشته نمی‌شوند.__

__==================================================__

__10. External Access__

__==================================================__

__دسترسی خارجی به عنوان بخشی از Business Partner ذخیره نمی‌شود.__

__مدل:__

__Business Partner__

__|__

__External Access Mapping__

__|__

__Identity Layer__

__|__

__External User Account__

__این قابلیت امکان ایجاد موارد زیر را در آینده فراهم می‌کند:__

- __Supplier Portal__
- __Customer Portal__
- __Contractor Portal__

__==================================================__

__11. ارتباط با ماژول‌های ERP__

__==================================================__

__Business Partner توسط ماژول‌های زیر مصرف می‌شود:__

__Sales__

__Customer__

__|__

__Sales Order__

__|__

__Invoice__

__Purchase__

__Supplier__

__|__

__Purchase Order__

__|__

__Receipt__

__Manufacturing__

__Contractor / Service Provider__

__|__

__Production Operation__

__Accounting__

__Customer Accounting Profile__

__Supplier Accounting Profile__

__Accounting Profile توسط Accounting Module مدیریت خواهد شد.__

__==================================================__

__12. اصول معماری نهایی__

__==================================================__

__تصمیمات قفل شده:__

__✓ Business Partner جزو ERP.Core Master Data است.__

__✓ هر Tenant مالک Business Partnerهای خود است.__

__✓ Global Business Directory در نسخه فعلی وجود ندارد.__

__✓ Business Partner قابلیت چند Role دارد.__

__✓ Person و Organization پشتیبانی می‌شوند.__

__✓ Role از Profile جدا است.__

__✓ Contact Person موجودیت مستقل است.__

__✓ External Access قابلیت مستقل روی Identity Layer است.__

__✓ Relationshipهای Business Partner Tenant-Owned هستند.__

__✓ Tenant Relationship در آینده به عنوان قابلیت SaaS جدا قابل بررسی است.__

__==================================================__

__وضعیت__

__Business Partner Architecture v1.0__

__Status: Locked__

