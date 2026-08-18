# سند رسمی ایندکس و ساختار درختواره مستندات HamarehERP

- **Version:** 1.0
- **Last Updated:** 2026-08-18
- **Category:** General Documentation
- **Status:** Draft / Approved
- **Source:** HamarehERP Architecture Documentation

---

__سند رسمی ایندکس و ساختار درختواره مستندات HamarehERP__

__۱. لایه حاکمیت معماری و خطوط قرمز (01_Architecture_Governance)__

- __تعریف:__ قوانین صلب، مفاهیم مشترک و استانداردهای نام‌گذاری پلتفرم.
	- __.__ 01_Architecture_Governance/ERP SaaS Architecture Foundation Rules.pdf
	- __.__ 01_Architecture_Governance/ERP SaaS Shared Language.pdf
	- __.__ 01_Architecture_Governance/Naming Convention Standard.pdf

__۲. نقشه‌های کلان لایه‌ها و برنامه‌های راهبردی (02_System_Blueprint_&_Roadmaps)__

- __تعریف:__ مدل‌های مفهومی، محدوده دامنه‌ها، وابستگی‌های ماژولار و برنامه‌های فازبندی.
	- __۲.۱.__ 02_System_Blueprint_&_Roadmaps/ERP SaaS Database Design Roadmap Phase 1.docx
	- __۲.۲.__ 02_System_Blueprint_&_Roadmaps/ERP SaaS Database Design Roadmap Phase 1.pdf
	- __۲.۳.__ 02_System_Blueprint_&_Roadmaps/ERP SaaS Development Roadmap.docx
	- __۲.۴.__ 02_System_Blueprint_&_Roadmaps/ERP SaaS Development Roadmap.pdf
	- __۲.۵.__ 02_System_Blueprint_&_Roadmaps/ERP SaaS Module Architecture Map v1.docx
	- __۲.۶.__ 02_System_Blueprint_&_Roadmaps/ERP SaaS Module Architecture Map v1.pdf
	- __۲.۷.__ 02_System_Blueprint_&_Roadmaps/ERP SaaS Module Boundary Definition v1.docx
	- __۲.۸.__ 02_System_Blueprint_&_Roadmaps/ERP SaaS Module Boundary Definition v1.pdf
	- __۲.۹.__ 02_System_Blueprint_&_Roadmaps/ERP SaaS Module Dependency Map v1.docx
	- __۲.۱۰.__ 02_System_Blueprint_&_Roadmaps/ERP SaaS Module Dependency Map v1.pdf
	- __۲.۱۱.__ 02_System_Blueprint_&_Roadmaps/ERP SaaS System Architecture Blueprint.docx
	- __۲.۱۲.__ 02_System_Blueprint_&_Roadmaps/ERP SaaS System Architecture Blueprint.pdf

__۳. استانداردهای فنی و زیرساختی پلتفرم (03_Technical_Infrastructure_Standards)__

- __تعریف:__ استانداردهای پیاده‌سازی ساس، قراردادهای ارتباطی API، معماری رویدادمحور و تک‌استک فنی.
	- __۳.۱.__ 03_Technical_Infrastructure_Standards/API Architecture Standard.docx
	- __۳.۲.__ 03_Technical_Infrastructure_Standards/API Architecture Standard.pdf
	- __۳.۳.__ 03_Technical_Infrastructure_Standards/ERP SaaS Backend Architecture.docx
	- __۳.۴.__ 03_Technical_Infrastructure_Standards/ERP SaaS Backend Architecture.pdf
	- __۳.۵.__ 03_Technical_Infrastructure_Standards/ERP SaaS Frontend Architecture.docx
	- __۳.۶.__ 03_Technical_Infrastructure_Standards/ERP SaaS Frontend Architecture.pdf
	- __۳.۷.__ 03_Technical_Infrastructure_Standards/ERP SaaS Frontend Integration Architecture.docx
	- __۳.۸.__ 03_Technical_Infrastructure_Standards/ERP SaaS Frontend Integration Architecture.pdf
	- __۳.۹.__ 03_Technical_Infrastructure_Standards/Event Architecture and Event Bus Standard.docx
	- __۳.۱۰.__ 03_Technical_Infrastructure_Standards/Event Architecture and Event Bus Standard.pdf
	- __۳.۱۱.__ 03_Technical_Infrastructure_Standards/Tenant Isolation Architecture Standard.docx
	- __۳.۱۲.__ 03_Technical_Infrastructure_Standards/Tenant Isolation Architecture Standard.pdf
	- __۳.۱۳.__ 03_Technical_Infrastructure_Standards/Workflow Architecture Standard.docx
	- __۳.۱base.__ 03_Technical_Infrastructure_Standards/Workflow Architecture Standard.pdf

__۴. لایه زیرساخت و بیزینس پلتفرم ابری (04_SaaS_Core_Platform_Layers)__

- __تعریف:__ سیستم حاکمیتی ابر، مدیریت اشتراک‌ها، پکیج‌ها و تراکنش‌های مالی پلتفرم (مستقل از ماژول‌های ERP).

__۴.الف. زیرپوشه مدیریت بیزینس ساس (Layer_1_SaaS_Business)__

- __۴.الف.۱.__ 04_SaaS_Core_Platform_Layers/Layer_1_SaaS_Business/Database Layer 1 - SaaS Business Layer.docx
- __۴.الف.۲.__ 04_SaaS_Core_Platform_Layers/Layer_1_SaaS_Business/Database Layer 1 - SaaS Business Layer.pdf

__۴.ب. زیرپوشه امنیت و ادمین فنی ساس (Layer_2_SaaS_Admin)__

- __۴.ب.۱.__ 04_SaaS_Core_Platform_Layers/Layer_2_SaaS_Admin/Database Layer 2 - SaaS Admin Layer.docx
- __۴.ب.۲.__ 04_SaaS_Core_Platform_Layers/Layer_2_SaaS_Admin/Database Layer 2 - SaaS Admin Layer.pdf

__۴.ج. زیرپوشه شبکه همکاران و افیلیت (Layer_3_Partner_Affiliate)__

- __۴.ج.۱.__ 04_SaaS_Core_Platform_Layers/Layer_3_Partner_Affiliate/Database Layer 3 - Partner Layer.docx
- __۴.ج.۲.__ 04_SaaS_Core_Platform_Layers/Layer_3_Partner_Affiliate/Database Layer 3 - Partner Layer.pdf

__۵. هویت متمرکز و داده‌های پایه یکپارچه (05_Identity_&_Master_Data_Layer)__

- __تعریف:__ فونداسیون افقی سیستم شامل لایه ۴ (هویت سازمانی) و لایه ۵ (۲۳ جدول پایه مشترک).

__۵.الف. زیرپوشه هسته هویت و دسترسی‌ها (Layer_4_Identity_Core)__

- __۵.الف.۱.__ 05_Identity_&_Master_Data_Layer/Layer_4_Identity_Core/Database Layer 4 - ERP Core Identity Organization Layer.docx
- __۵.الف.۲.__ 05_Identity_&_Master_Data_Layer/Layer_4_Identity_Core/Database Layer 4 - ERP Core Identity Organization Layer.pdf

__۵.ب. زیرپوشه جداول پایه و تراکنشی مشترک (Layer_5_Master_Data_Tables)__

- __۵.ب.۱.__ 05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/02_Master_Data_Table_Definitions.docx
- __۵.ب.۲.__ 05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/02_Master_Data_Table_Definitions.pdf
- __۵.ب.۳.__ 05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/Business Partner Architecture Standard v1.0.docx
- __۵.ب.۴.__ 05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/Business Partner Architecture Standard v1.0.pdf
- __۵.ب.۵.__ 05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/ERP SaaS Master Data Architecture Standard v1.1.docx
- __۵.ب.۶.__ 05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/ERP SaaS Master Data Architecture Standard v1.1.pdf

__۶. ماژول‌های عمودی بیزینسی (06_Vertical_ERP_Modules)__

- __تعریف:__ دامنه‌های عملیاتی و مستقل بیزینس بر پایه DDD بدون ارتباط فیزیکی کلید خارجی.

__۶.۱. زیرپوشه موتور گردش کار پورتال (01_Workflow_Engine_Portal)__

- __۶.۱.۱.__ 06_Vertical_ERP_Modules/01_Workflow_Engine_Portal/ADD_04_Dynamic_Multi_Tenant_Workflow_Engine.docx
- __۶.۱.۲.__ 06_Vertical_ERP_Modules/01_Workflow_Engine_Portal/ADD_04_Dynamic_Multi_Tenant_Workflow_Engine.pdf

__۶.۲. زیرپوشه انبار و لجستیک (02_Inventory_&_Logistics)__

- __۶.۲.۱.__ 06_Vertical_ERP_Modules/02_Inventory_&_Logistics/ADD_05_Inventory_Domain_Architecture.docx
- __۶.2.۲.__ 06_Vertical_ERP_Modules/02_Inventory_&_Logistics/ADD_05_Inventory_Domain_Architecture.pdf
- __۶.۲.۳.__ 06_Vertical_ERP_Modules/02_Inventory_&_Logistics/Inventory_Logistics_Module.docx
- __۶.۲.۴.__ 06_Vertical_ERP_Modules/02_Inventory_&_Logistics/Inventory_Logistics_Module.pdf

__۶.۳. زیرپوشه بازرگانی خرید و فروش (03_Procurement_&_Sales)__

- __۶.۳.۱.__ 06_Vertical_ERP_Modules/03_Procurement_&_Sales/ADD_06_Procurement_Sales_Architecture.docx
- __۶.۳.۲.__ 06_Vertical_ERP_Modules/03_Procurement_&_Sales/ADD_06_Procurement_Sales_Architecture.pdf
- __۶.۳.۳.__ 06_Vertical_ERP_Modules/03_Procurement_&_Sales/Procurement_Sales_Table_Definitions.docx
- __۶.۳.۴.__ 06_Vertical_ERP_Modules/03_Procurement_&_Sales/Procurement_Sales_Table_Definitions.pdf

__۶.۴. زیرپوشه حسابداری مالی (04_Financial_Accounting)__

- __۶.۴.۱.__ 06_Vertical_ERP_Modules/04_Financial_Accounting/ADD_07_Financial_Accounting_Architecture.docx
- __۶.۴.۲.__ 06_Vertical_ERP_Modules/04_Financial_Accounting/fin_accounting_table_definitions.docx

__۶.۵. زیرپوشه تولید و فرمولاسیون (05_Manufacturing_&_Production)__

- __۶.۵.۱.__ 06_Vertical_ERP_Modules/05_Manufacturing_&_Production/ADD_08_Manufacturing_Production_Architecture.docx
- __۶.۵.۲.__ 06_Vertical_ERP_Modules/05_Manufacturing_&_Production/ADD_08_Manufacturing_Production_Architecture.pdf
- __۶.۵.۳.__ 06_Vertical_ERP_Modules/05_Manufacturing_&_Production/mfg_production_table_definitions.docx
- __۶.۵.۴.__ 06_Vertical_ERP_Modules/05_Manufacturing_&_Production/mfg_production_table_definitions.pdf

__۷. نیازمندی‌های محیط توسعه و نصاب‌ها (07_Environment_&_Tools)__

- __تعریف:__ الزامات سرور محلی و پکیج‌های پیش‌نیاز.
	- __۷.۱.__ 07_Environment_&_Tools/Local Development Requirements.docx
	- __۷.۲.__ 07_Environment_&_Tools/Local Development Requirements.pdf

__۷.الف. زیرپوشه فایل‌های نصب (Installers)__

- __۷.الف.۱.__ 07_Environment_&_Tools/Installers/CursorUserSetup-x64-3.10.20.exe
- __۷.الف.۲.__ 07_Environment_&_Tools/Installers/Docker Desktop Installer.exe
- __۷.الف.۳.__ 07_Environment_&_Tools/Installers/Git-2.55.0.2-64-bit.exe
- __۷.الف.۴.__ 07_Environment_&_Tools/Installers/postgresql-18.4-2-windows-x64.exe

