# FE-P0 Foundation & Shell — Phase Closure v1.0

- **Version:** 1.0
- **Date:** 2026-09-13
- **Status:** Closed (pending Product Owner sign-off)
- **Repos:** hamarehSaasErp-Front
- **Related:** Frontend_Phase_Kickoff_Decision_Record_v1.0.md

---

## 1. Exit criteria (Definition of Done)

| معیار | وضعیت |
|--------|--------|
| Login با credentials واقعی Backend (password + OTP) | Done |
| درخواست‌های محافظت‌شده با Bearer + `X-Tenant-ID` | Done (apiClient) |
| `/dashboard/*` بدون Session قابل دسترسی نیست | Done (AuthGuard) |
| کاربر لاگین‌شده از `/login` به داشبورد هدایت می‌شود | Done (GuestGuard) |
| Shell (Header + Sidebar) نام کاربر/نقش/tenant و Logout | Done |
| RTL + Mobile drawer برای Shell | Done |
| UI Guide (UI-00..UI-11) در دسترس و بدون نقض قوانین | Done |
| الگوهای مشترک آماده ماژول (PageHeader, EmptyState, Can, ErrorBoundary, module scaffold) | Done |

---

## 2. Sprint summary

### Sprint 1 — Blockers (T01–T06)

- Central API Client (`src/api/client.ts`)
- Auth types / store / service / TenantProvider
- AuthGuard + IdleLock
- Login page (password + OTP UX, human gate on threshold)

### Sprint 2 — Shell ready for modules

- Header/Sidebar with session data + Logout
- GuestGuard for public auth routes
- `Can` / `usePermission` (UI-only; Backend remains SoT)
- `PageHeader`, `EmptyState`, `AppErrorBoundary`
- Mobile navigation drawer
- Module scaffold (`src/modules/_template`) + Organization placeholder page

### Sprint 3 — Quality & close

- Dashboard home shows live session snapshot
- Organization route wired under shell
- This closure checklist + known debt list

---

## 3. Known debt (accepted for post-P0)

1. **Refresh token** — Backend still Sanctum personal token without refresh endpoint; 401 clears session only.
2. **Tenant display name** — Header shows tenant UUID until a tenant profile endpoint is consumed.
3. **Sidebar nav permissions** — Static menu; filter by `hasPermission` / module flags when Backend exposes entitlement list.
4. **Forgot-password flow** — UI link present; Backend recovery API not wired.
5. **Human-check** — Client-side slide only; rate-limit remains Backend responsibility.
6. **E2E automated tests** — Manual scenario only in P0; Playwright suite deferred.
7. **Search in header** — Visual placeholder; global search deferred to later phase.

---

## 4. Manual acceptance scenario

1. Open `/login` while logged out → form visible.
2. Login with valid password → `/dashboard` shows user + roles + tenant id.
3. Open protected API via any module call → request has `Authorization` + `X-Tenant-ID`.
4. Logout → redirected to `/login`; revisiting `/dashboard` redirects to login.
5. Login again → `/login` redirects to dashboard (GuestGuard).
6. OTP path: request code → enter digits → session or org picker.
7. After 3 bad password attempts, next **button click** shows human slide; after pass, login proceeds.
8. Mobile width: hamburger opens drawer; navigation works.
9. `/dashboard/ui-guide` and `/dashboard/organization` render inside shell.

---

## 5. Next phase recommendation

Per Kickoff Decision Record §4.3 module order:

1. **Organization** (Company / Branch / Department) — real CRUD against Layer 5 APIs
2. Master Data پایه
3. Identity tenant-facing screens (profile, roles display)
4. Inventory

---

**End of FE-P0 Closure Document**
