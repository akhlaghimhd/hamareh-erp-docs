# Project Debt Register v1.0

**Document ID:** DEBT-REGISTER-v1.0  
**SSOT for open project debts across all layers/modules**  
**Created:** 2026-10-06  
**Last updated:** 2026-10-06T19:50:00+02:00  
**Repos:** akhlaghimhd/hamareh-erp-docs  
**Related status docs (do not duplicate; link only):**
- `05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/ORG_Smart_Hierarchy_Status_and_Debt_v1.0.md`
- `05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/ORG_Intercompany_Status_and_Debt_v1.0.md`
- `05_Identity_&_Master_Data_Layer/Layer_5_Master_Data_Tables/ORG_Sales_Purch_Status_and_Debt_v1.0.md`
- `04_SaaS_Core_Platform_Layers/Layer_2_SaaS_Admin/ADR-SAASADM-001_Feature_Pack_Model_v1.0.md`

---

## Register law (LOCKED 2026-10-06)

Every new debt entry **must** include all of the following fields, in this order:

1. **Module** — official layer/module (e.g. IdentityCore, Organization)
2. **Section** — sub-area inside the module
3. **Reason** — why the debt was created (checkable when closing)
4. **Layer type** — Backend | Frontend | Architecture (or combination)
5. **Created at** — ISO date-time of registration
6. **Owner decision** — explicit product/engineering decision (or «ثبت اولیه — منتظر تصمیم»)
7. **Suggestion** — recommended approach / unblock path

**Rules:**
- Do not delete closed debts; mark **Status: CLOSED** with close date and verification note.
- Prefer linking to existing per-topic Status_and_Debt files for deep detail; this register is the **index + decision log**.
- When user says «بدهی را ثبت کن», append here with the 7 fields above.
- Commit messages English; narrative for user in Persian.

**Status values:** OPEN | BLOCKED | PARTIAL | DEFERRED | CLOSED

---

PLACEHOLDER_FULL_FILE
