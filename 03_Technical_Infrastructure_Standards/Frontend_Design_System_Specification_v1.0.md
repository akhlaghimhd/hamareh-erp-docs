# Frontend Design System Specification v1.0

- **Version:** 1.0
- **Date:** 2026-09-10
- **Status:** Locked / Non-Negotiable (Structure)
- **Category:** Technical Infrastructure Standards – Frontend
- **Related Documents:**
  - Frontend_Phase_Kickoff_Decision_Record_v1.0.md
  - ERP SaaS Frontend Architecture.md
  - ERP SaaS Frontend Integration Architecture.md
  - Backend_Phase_Closure_Report_L1_to_L6_v1.0.md
- **Figma File:** https://www.figma.com/design/RykextIEfCWahPyaqEnAuZ (Hamareh ERP – Design System)

---

## 1. Purpose

این سند مشخصات کامل و رسمی Design System پلتفرم Hamareh ERP را تعریف می‌کند.

هدف:
- ایجاد مرجع واحد برای تمام تصمیمات بصری و تعاملی Frontend
- جلوگیری از پراکندگی و تصمیم‌گیری مجدد
- پشتیبانی از توسعه موازی سیاست‌گذاری و اجرا
- تضمین کیفیت سازمانی و مقیاس‌پذیری UI/UX

هرگونه انحراف از ساختار این سند بدون Architecture Amendment ممنوع است.

---

## 2. Official Repositories & Tools (Reminder)

| نوع | آدرس / ابزار |
|-----|-------------|
| Backend Code | https://github.com/akhlaghimhd/hamarehSaasErp.git |
| Frontend Code | https://github.com/akhlaghimhd/hamarehSaasErp-Front.git |
| Architecture Docs | https://github.com/akhlaghimhd/hamareh-erp-docs.git |
| Design Tool (Visual SSOT) | Figma – فایل Hamareh ERP – Design System |

---

## 3. Design System Structure (Complete Inventory)

### 3.1 Foundations
- Color System (Primary, Secondary, Success, Warning, Danger, Info + full scale 50–950 + Dark Mode)
- Typography (Vazirmatn – Heading 1–6, Body, Small, Caption, Label, Overline)
- Spacing (8-point scale + semantic tokens)
- Border Radius (none, sm, md, lg, xl, full)
- Elevation / Shadow (sm, md, lg, xl)
- Border tokens
- Iconography sizes (16, 20, 24) – Outline / Solid
- Motion tokens (duration & easing)
- Breakpoints (Mobile, Tablet, Desktop)

### 3.2 Base Components
- Button (Primary, Secondary, Outline, Ghost, Destructive) + sizes + Loading/Disabled
- Icon Button
- Input (Text, Password, Number, Search) + Label + Helper + Error + Prefix/Suffix
- Textarea
- Select / Combobox
- Checkbox
- Radio Group
- Switch
- Badge / Tag / Status Pill
- Avatar (image + initials fallback)
- Tooltip
- Divider
- Skeleton

### 3.3 Form Patterns
- Form Field (Label + Control + Helper + Error)
- Form Layout (single-column & two-column)
- Form Section / Fieldset
- Inline Validation State
- Required Indicator
- Form Actions (Save / Cancel)

### 3.4 Data Display
- Table (Header, Row, Cell, Sortable, Sticky, Empty, Loading, Row Actions)
- Pagination
- Data List
- Description List
- Key-Value Pair
- Status Indicator
- Empty State (generic)

### 3.5 Feedback & Messaging
- Alert (Success, Warning, Error, Info)
- Toast / Notification
- Banner
- Inline Message
- Progress / Spinner
- Confirmation Dialog

### 3.6 Navigation & Layout
- Sidebar (expanded / collapsed)
- Top Header / App Bar
- Breadcrumb
- Tabs
- Vertical Navigation Item
- Page Header (title + page actions)
- Content Layout (with Sidebar)

### 3.7 Overlays
- Modal / Dialog (sm, md, lg)
- Drawer (RTL – from right)
- Dropdown Menu
- Popover
- Command Palette (optional in later phase)

### 3.8 ERP-Specific Patterns
- Document Status Badge (Draft, Pending, Approved, Rejected, Posted)
- Amount / Currency Display
- Date & DateTime Display
- User Chip
- Permission Gate (disabled state for unauthorized actions)
- Worklist Item
- Filter Bar
- Bulk Actions Bar

### 3.9 Common States
- Default, Hover, Focus, Active/Selected, Disabled, Loading, Error, Empty

### 3.10 Branding & Multi-Tenant Identity
- Platform Branding (Hamareh logo, name, colors)
- Tenant Branding (customer logo, organization name, optional brand color)
- Logo Variants (full, collapsed, light/dark, favicon)
- App Name & Tagline
- Login / Auth Branding
- Favicon & Browser Meta

### 3.11 User & Identity Components
- User Avatar (image, initials, online status)
- User Menu (profile, settings, logout)
- User Chip / Mention
- Role Badge
- Permission Indicator
- Profile Header
- Session / Security Info

### 3.12 Reporting & Analytics Patterns
- Report Page Header
- Filter Panel
- KPI / Metric Card
- Chart Container
- Report Table (with totals & grouping)
- Date Range Picker
- Export Actions
- No Data State (Report)
- Print-friendly Layout

### 3.13 Complementary Patterns
- Command Palette / Global Search
- Help & Documentation Trigger
- Onboarding / Empty Workspace
- Notification Center
- Activity Timeline
- File / Attachment Item
- Comment Thread (future)

### 3.14 Accessibility (Non-Negotiable)
- Focus States (consistent ring)
- Color Contrast (WCAG AA minimum)
- Keyboard Navigation (Tab order, Escape, Enter)
- Screen Reader Labels (aria)
- Reduced Motion support
- Error Identification linked to fields

### 3.15 Density & Layout Modes
- Comfortable Density (default)
- Compact Density (for power users & dense tables)
- Content Max-Width
- Sticky Zones (table header, filter bar, action bar)

### 3.16 Micro-interactions & Feedback
- Button Loading State
- Success Feedback
- Destructive Action Flow (two-step confirmation)
- Hover & Press States
- Transition Tokens

### 3.17 Content & Microcopy Guidelines
- Voice & Tone (formal, clear, unambiguous)
- Button Labels (action-oriented)
- Empty State Copy (actionable)
- Error Messages (problem + possible solution)
- Confirmation Texts (standard delete/cancel/leave)

### 3.18 First-time & Empty Experiences
- First Login / Welcome
- Empty Module State
- Onboarding Checklist (optional)
- No Permission State

### 3.19 Global Behavior Patterns
- Toast Stacking & Positioning
- Unsaved Changes Guard
- Optimistic UI Rules
- Pagination vs Infinite Scroll (Pagination preferred for ERP)
- Default Sorting & Filtering behavior

### 3.20 Documentation & Governance
- Do’s and Don’ts per component
- Usage Guidelines (Modal vs Drawer, etc.)
- Component Status (Ready / Draft / Deprecated)
- Design System Versioning

---

## 4. Implementation Priority (Phased)

### Phase A – Immediate (Current Focus)
Foundations (complete) + Button + Input + Form Field + Badge + Alert + Table (base) + Modal + Page Header + basic Branding tokens

### Phase B
Sidebar + Header + Tabs + Pagination + Empty State + Drawer + Select + Checkbox/Switch + User Menu + Avatar

### Phase C
Filter Bar, Bulk Actions, Worklist, Document Status, Reporting patterns, Density modes, advanced Accessibility documentation, Microcopy guidelines

---

## 5. Key Locked Decisions

| موضوع | تصمیم |
|------|--------|
| Font Family | **Vazirmatn** (primary) |
| UI Library (Code) | Shadcn/UI + Tailwind CSS |
| Direction | RTL first |
| Density | Comfortable default + Compact option |
| Table Strategy | Pagination (not infinite scroll) |
| Design Tool | Figma (Visual Single Source of Truth) |
| Approach | Hybrid (Foundations & key components in Figma → parallel implementation) |

---

## 6. Governance

- این سند از تاریخ ۲۰۲۶-۰۹-۱۰ از نظر **ساختار و دامنه** قفل است.
- جزئیات بصری هر کامپوننت در Figma به‌عنوان Visual SSOT نگهداری می‌شود.
- تغییرات ساختاری نیازمند Architecture Amendment است.
- اجرای کامپوننت‌ها باید با این فهرست و اولویت‌بندی هم‌خوان باشد.

---

## 7. Current Execution Status

| بخش | وضعیت |
|-----|--------|
| Foundations (Colors, Typography, Spacing, Radius) | Started in Figma |
| Phase A Components | Pending (next) |
| Code alignment (Shadcn) | Pending |

---

**End of Document**

**This specification is the Single Source of Truth for Hamareh ERP Design System scope and structure.**
