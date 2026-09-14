# FE-P0 Foundation & Shell — Phase Closure v1.1

- **Version:** 1.1
- **Date:** 2026-09-14
- **Status:** Closed (pending Product Owner sign-off on manual scenario T17)
- **Repos:** hamarehSaasErp-Front
- **Related:** Frontend_Phase_Kickoff_Decision_Record_v1.0.md

---

## 1. Exit criteria (Definition of Done)

| معیار | وضعیت |
|--------|--------|
| Login با credentials واقعی Backend (password + OTP) | Done |
| درخواست‌های محافظت‌شده با Bearer + `X-Tenant-ID` | Done (apiClient + limited retry) |
| `/dashboard/*` بدون Session قابل دسترسی نیست | Done (AuthGuard) |
| کاربر لاگین‌شده از `/login` به داشبورد هدایت می‌شود | Done (GuestGuard) |
| Shell (Header + Sidebar) نام کاربر/نقش/tenant و Logout | Done |
| RTL + Mobile drawer برای Shell | Done |
| UI Guide (UI-00..UI-11) در دسترس و بدون نقض قوانین | Done |
| الگوهای مشترک آماده ماژول (PageHeader, Breadcrumb, EmptyState, Can, ErrorBoundary, Form, DataTable, DirtyDialog, StatusChip, module scaffold) | Done |

---

## 2. Task checklist T01–T18 (full)

| کد | عنوان | وضعیت |
|----|--------|--------|
| T01 | API Client مرکزی | **Done** (Bearer, X-Tenant-ID, timeout, error normalize, limited retry on network/502–504). Refresh queue deferred — Backend has no refresh endpoint. |
| T02 | Auth Store + Session (Zustand) | **Done** |
| T03 | Auth Service Layer | **Done** (password, OTP, select-tenant, logout). Me/Refresh when Backend adds endpoints. |
| T04 | صفحه Login | **Done** |
| T05 | Route Protection / Auth Guard | **Done** (+ GuestGuard, IdleLock) |
| T06 | Tenant Context Provider | **Done** |
| T07 | تکمیل App Shell | **Done** (+ Breadcrumb pattern on PageHeader) |
| T08 | ساختار پوشه‌ای ماژول‌محور | **Done** |
| T09 | Shared Form Primitives | **Done** (`Form`, `FormField`, `FormItem`, `FormLabel`, `FormControl`, `FormMessage`, `FormGrid`) |
| T10 | Shared Table / Data Display | **Done** (`DataTable` with page-size, empty-initial vs empty-search, skeleton) |
| T11 | Overlay Dirty-Lock | **Done** (`DirtyDialog` — Escape/backdrop/X blocked while dirty) |
| T12 | Feedback استاندارد | **Done** (Toast, Alert, Skeleton, EmptyState, StatusChip) |
| T13 | Theme / Design Token Runtime | **Done** (base; advanced tenant branding remains post-P0 polish) |
| T14 | Accessibility پایه Shell | **Done** (focus-visible global, aria-label on icon controls, prefers-reduced-motion) |
| T15 | Environment & Config | **Done** |
| T16 | Error Boundary + Global Error UX | **Done** |
| T17 | تست دستی سناریوی کامل | **Documented** — requires PO run against live Backend |
| T18 | سند بسته شدن FE-P0 | **Done** (this document v1.1) |

---

## 3. Sprint summary

### Sprint 1 — Blockers (T01–T06)

- Central API Client (`src/api/client.ts`)
- Auth types / store / service / TenantProvider
- AuthGuard + IdleLock + GuestGuard
- Login page (password + OTP UX, human gate on threshold)

### Sprint 2 — Shell ready for modules (T07–T12)

- Header/Sidebar with session data + Logout + mobile drawer
- `Can` / `usePermission`
- `PageHeader` + `Breadcrumb`, `EmptyState`, `AppErrorBoundary`
- Module scaffold + Organization placeholder
- **Form primitives**, **DataTable**, **DirtyDialog**, **StatusChip**

### Sprint 3 — Quality & close (T13–T18)

- Theme tokens + reduced-motion + focus-visible
- Dashboard session snapshot + Organization route
- Closure checklist + known debt list

---

## 4. Known debt (accepted for post-P0 — Backend or later phase)

1. **Refresh token** — Backend still Sanctum personal token without refresh endpoint; 401 clears session only.
2. **Tenant display name** — Header shows tenant UUID until a tenant profile endpoint is consumed.
3. **Sidebar nav permissions** — Static menu; filter by `hasPermission` / module flags when Backend exposes entitlement list.
4. **Forgot-password flow** — UI link present; Backend recovery API not wired.
5. **Human-check** — Client-side slide only; rate-limit remains Backend responsibility.
6. **E2E automated tests** — Manual scenario only in P0; Playwright suite deferred.
7. **Search in header** — Visual placeholder; global search deferred to later phase.
8. **Advanced tenant branding runtime** — Design tokens base done; full white-label runtime later.

---

## 5. Manual acceptance scenario (T17)

1. Open `/login` while logged out → form visible.
2. Login with valid password → `/dashboard` shows user + roles + tenant id.
3. Open protected API via any module call → request has `Authorization` + `X-Tenant-ID`.
4. Logout → redirected to `/login`; revisiting `/dashboard` redirects to login.
5. Login again → `/login` redirects to dashboard (GuestGuard).
6. OTP path: request code → enter digits → session or org picker.
7. After 3 bad password attempts, next **button click** shows human slide; after pass, login proceeds.
8. Mobile width: hamburger opens drawer; navigation works.
9. `/dashboard/ui-guide` and `/dashboard/organization` render inside shell.
10. (Patterns) Form error under field, DirtyDialog blocks Escape when dirty, DataTable empty-search ≠ empty-initial.

---

## 6. Next phase recommendation

Per Kickoff Decision Record §4.3 module order:

1. **Organization** (Company / Branch / Department) — real CRUD against Layer 5 APIs
2. Master Data پایه
3. Identity tenant-facing screens (profile, roles display)
4. Inventory

> Note: Any FE-P1/FE-P2 numbering that places Identity before Organization conflicts with the Locked Kickoff order and requires Architecture Amendment.

---

**End of FE-P0 Closure Document v1.1**
