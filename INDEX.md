# 🏗️ HamarehERP Architecture Documentation Index

This repository contains the official architecture, database design, and technical standards for the HamarehERP SaaS Platform.

## ⚠️ Layer numbering SSOT

**Official layer numbers and code-module mapping:**  
[ADD_Layer_Module_Code_Mapping_v1.0](./02_System_Blueprint_&_Roadmaps/ADD_Layer_Module_Code_Mapping_v1.0.md)

Do **not** use superseded numbering (e.g. Identity as Layer 2 from older Blueprint drafts).

| Layer | Name | Primary code module |
|-------|------|---------------------|
| 1 | SaaS Platform Business | `SaasPlatform` |
| 2 | SaaS Admin | `SaasAdmin` |
| 3 | Partner & Affiliate | partner_layer |
| 4 | Identity & Access Core | `IdentityCore` |
| 5 | ERP Foundation | `Organization`, `MasterData`, `DocumentManagement` |
| 6 | ERP Business Modules | vertical modules |
| 7 | Extensions & Integrations | future |

## 📚 Documentation Map

### 1. Architecture Governance & Foundation
- [ERP SaaS Architecture Foundation Rules](./01_Architecture_Governance/ERP%20SaaS%20Architecture%20Foundation%20Rules.md)
- [ERP SaaS Shared Language](./01_Architecture_Governance/ERP%20SaaS%20Shared%20Language.md)
- [Naming Convention Standard](./01_Architecture_Governance/Naming%20Convention%20Standard.md)

### 2. System Blueprint & Roadmaps
- [**ADD Layer / Module / Code Mapping (SSOT)**](./02_System_Blueprint_&_Roadmaps/ADD_Layer_Module_Code_Mapping_v1.0.md)
- [Consolidated Blueprint v2.1](./02_System_Blueprint_&_Roadmaps/ERP%20SaaS%20Architecture%20Consolidated%20Blueprint%20v2.0.md)
- [Module Architecture Map v1.1](./02_System_Blueprint_&_Roadmaps/ERP%20SaaS%20Module%20Architecture%20Map%20v1.md)
- [System Architecture Blueprint](./02_System_Blueprint_&_Roadmaps/ERP%20SaaS%20System%20Architecture%20Blueprint.md)
- Other maps and roadmaps in [02_System_Blueprint_&_Roadmaps](./02_System_Blueprint_&_Roadmaps/)

### 3. Technical Infrastructure Standards
- [Backend Architecture](./03_Technical_Infrastructure_Standards/ERP%20SaaS%20Backend%20Architecture.md)
- [Frontend Architecture](./03_Technical_Infrastructure_Standards/ERP%20SaaS%20Frontend%20Architecture.md)
- [API & Event Architecture](./03_Technical_Infrastructure_Standards/)

### 4. SaaS Core Platform Layers (Database)
- [Layer 1: SaaS Platform Business (folder alias: SaaS Business)](./04_SaaS_Core_Platform_Layers/Layer_1_SaaS_Business/Database%20Layer%201%20-%20SaaS%20Business%20Layer.md)
- [Layer 2: SaaS Admin](./04_SaaS_Core_Platform_Layers/Layer_2_SaaS_Admin/Database%20Layer%202%20-%20SaaS%20Admin%20Layer.md)
- [Layer 3: Partner & Affiliate](./04_SaaS_Core_Platform_Layers/Layer_3_Partner_Affiliate/Database%20Layer%203%20-%20Partner%20Layer.md)

### 5. Identity & Master Data / Foundation
- [Layer 4: Identity & Access Core](./05_Identity_&_Master_Data_Layer/Layer_4_Identity_Core/Database%20Layer%204%20-%20ERP%20Core%20Identity%20Organization%20Layer.md) *(file title legacy; content scope = Identity; Organization is Layer 5)*
- [Layer 5 Master Data](./05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/ERP%20SaaS%20Master%20Data%20Architecture%20Standard%20v1.1.md)

### 6. Vertical ERP Modules (Layer 6)
- [Workflow Engine](./06_Vertical_ERP_Modules/01_Workflow_Engine_Portal/)
- [Inventory & Logistics](./06_Vertical_ERP_Modules/02_Inventory_&_Logistics/)
- [Procurement & Sales](./06_Vertical_ERP_Modules/03_Procurement_&_Sales/)
- [Financial Accounting](./06_Vertical_ERP_Modules/04_Financial_Accounting/)
- [Manufacturing & Production](./06_Vertical_ERP_Modules/05_Manufacturing_&_Production/)

---
*Last Updated: 2026-08-29*
