# UI Restoration Playbook — Identity & Organization v1.0

| Field | Value |
|-------|--------|
| **Status** | LOCKED — binding on agent |
| **Date** | 2026-10-19 |
| **Scope** | Front `hamarehSaasErp-Front` develop — modules Identity + Organization |
| **Trigger** | User observed visual regressions after security fixes; agent must restore approved UI without reopening security holes |

---

## ۰. اصل مطلق (غیرقابل نقض)

1. **Backend-only fixes never touch FE** unless user explicitly says «UI را هم تغییر بده».
2. **لیست/جدول (members, roles, scopes, role-assignment, branches, departments, companies, …) هرگز full-rewrite نمی‌شوند** برای باگ نامرتبط. فقط minimal line-diff.
3. **قبل از هر ویرایش UI:** نسخهٔ سالم از git history / baseline تأییدشده خوانده شود — نه حدس از artifacts.
4. **امکانات تأییدشده قبلی باید برگردند**؛ حذف خاموش قابلیت ممنوع.
5. **باگ‌های امنیتی بسته‌شده نباید برگردند** (هلدینگ، سقف نقش/اسکوپ، sibling، ghost role، HTTP 403).
6. **کار صفحه به صفحه است** — یک PR/commit منطقه‌ای؛ نه یک بازنویسی سراسری.
7. **تحویل فقط از طریق git push به develop** — مسیر sandbox/artifacts به کاربر داده نشود.

---

## ۱. هدف این دستورالعمل

وقتی کاربر گفت «ظاهر خراب شده / امکانات کم شده / UI را درست کن»:

1. baseline تأییدشدهٔ هر صفحه را پیدا کن  
2. diff با نسخهٔ فعلی بساز  
3. **فقط** اختلاف‌های ناخواسته را برگردان  
4. نگه دار: fixهای امنیتی/منطقی لازم  
5. چک‌لیست UI/UX فارسی را روی همان صفحه پاس کن  
6. لیست تغییرات همان صفحه را به کاربر گزارش کن **قبل از** (یا همزمان با) push — بدون پنهان‌کاری

---

## ۲. یافتن baseline (نسخهٔ تأیید UI)

### ۲.۱ ترتیب اولویت منبع حقیقت

1. **کامیت صریح کاربر** که گفته «این UI اوکی است» (اگر در چت ثبت شده).
2. **`origin/main` Front** اگر صفحه روی main پایدار است.
3. **آخرین کامیت `develop` قبل از موج security/fix ناخواسته** که:
   - فایل لیست/جزئیات کامل است (نه stub/PLACEHOLDER/70 بایت re-export خالی)
   - DataTable + Drawer + bulk + Permission Gate دارد
4. **فایل‌های RESTORE/FULL در history** فقط اگر git نشان دهد محتوا با baseline یکی است — اولویت با git است نه حدس.

### ۲.۲ دستورات اجباری قبل از ویرایش هر صفحه

```powershell
cd <Front-repo>
git fetch origin
git log origin/develop --oneline -- <path-to-page-file> | Select-Object -First 20
git log origin/main --oneline -- <path-to-page-file> | Select-Object -First 10
git show <baseline-sha>:<path-to-page-file> | Select-Object -First 5
# اندازه فایل: stub مشکوک اگر خیلی کوچک باشد
(Get-Item <path>).Length
```

### ۲.۳ علامت خطر (فایل خراب / بازنویسی‌شده)

- اندازهٔ غیرطبیعی کوچک (مثلاً members-list-body ~70 bytes فقط re-export)
- حذف DataTable / bulk / Drawer / export
- متن انگلیسی خام به‌جای فارسی
- از بین رفتن `toFaDigits` / تاریخ شمسی
- URL با id در path (نقض قانون URL)

---

## ۳. کاتالوگ صفحات (کار صفحه به صفحه)

برای **هر ردیف**: baseline را جداگانه پیدا کن → diff → لیست delta → restore/minimal fix → self-check.

### ۳.۱ Identity

| صفحه / سطح | مسیر اصلی (تقریبی) | امکانات تأییدشدهٔ مورد انتظار (حداقلی) |
|------------|---------------------|------------------------------------------|
| Hub هویت | `identity-home.tsx` | کارت‌ها، لینک، gate |
| لیست اعضا | `members-list.tsx` → `members-list-impl.tsx` + helpers | DataTable compact، search/filter/paging، bulk، Create Drawer، export، company filter هلدینگ، soft-delete bucket، Permission Gate، اعداد فارسی |
| جزئیات عضو | `member-detail*.tsx` + AssignRoles/Scopes cards | focus+detail (نه id در URL)، نقش‌ها (pending dimmed)، اسکوپ‌ها، وضعیت، history، بدون hard redirect |
| ایجاد عضو | `member-create-drawer.tsx` | Drawer راست، dirty-guard، موبایل اجباری، بدون password در create |
| لیست نقش‌ها | `roles-list.tsx` + tree | جدول + درخت، bulk، Create Drawer، parent select |
| جزئیات نقش | `role-detail.tsx` | مجوزها، درخت، stay-on-page بعد create |
| مجوزها | `permissions-list.tsx` | لیست فارسی، گروه‌بندی، بدون نمایش code خام اگر محصول مخفی کرده |
| اسکوپ‌ها | `scopes-list.tsx` | CRUD/لیست، ceiling-aware اگر قبلاً بوده |
| صف تأیید نقش | `role-assignment-requests-list.tsx` | فقط PENDING، بدون Create، ستون‌های قفل محصول |
| پروفایل من | `profile-me.tsx` | MFA/change-password در صورت وجود endpoint |
| Access Cert / Privileged / SoD | صفحات مربوط | مطابق sprint قبلی — حذف خاموش ممنوع |

### ۳.۲ Organization

| صفحه | مسیر تقریبی | امکانات مورد انتظار |
|------|-------------|---------------------|
| Hub سازمان | hub | feature-pack gating مستقل |
| لیست شرکت‌ها | companies list | bulk، primary rules، soft-delete |
| جزئیات شرکت | company detail + focus URL | branches/depts panels در صورت baseline غنی، ref_code اگر تأیید شده |
| لیست شعب | branches list | bulk مثل شرکت‌ها، فیلتر شرکت |
| لیست دپارتمان‌ها | departments list | مشابه |
| ساختار فروش/خرید / IC | در صورت وجود در FE | gate با pack |

---

## ۴. چک‌لیست UI/UX اجباری (هر صفحه بعد از ویرایش)

عامل **قبل از اعلام تمام** باید این موارد را خود مرور کند (کد + در صورت امکان lint/typecheck):

| # | مورد |
|---|------|
| 1 | RTL / راست‌چین فرم و جدول |
| 2 | فاصله از لبه (padding صفحه مطابق الگوی پروژه، نه چسبیده به لبه) |
| 3 | عرض حداقل ۳۶۰px؛ Drawer موبایل full-screen |
| 4 | متن UI فارسی؛ بدون جمله انگلیسی خام |
| 5 | اعداد با `toFaDigits` |
| 6 | تاریخ شمسی در نمایش |
| 7 | DataTable compact؛ empty/loading/error فارسی |
| 8 | Toast فارسی؛ confirm برای حذف |
| 9 | Dirty-guard روی Drawer |
| 10 | Permission Gate: دکمه فقط اگر مجوز هست |
| 11 | بدون id دیتابیس در path مرورگر |
| 12 | امکانات baseline حفظ شده (bulk، export، filter، …) |

---

## ۵. امنیت — نباید دوباره بشکنند

هنگام restore UI:

| قانون | رفتار UI |
|--------|----------|
| ADR-003 sibling | فیلتر شرکت / عدم نمایش عضو خارج سقف |
| ADR-005 ceiling | picker اسکوپ فقط در سقف |
| ADR-006 role grant | پیام خطای API فارسی؛ عدم پنهان کردن 403 |
| soft-delete نقش | فقط نقش active در chip |
| HTTP assign | همان endpoint؛ دستکاری client-side برای دور زدن سقف ممنوع |

تست BE موجود (ماتریس سرویس + HTTP) بعد از هر موج FE مرتبط باید همچنان سبز بماند:

```powershell
docker compose exec app php artisan test --filter=HoldingAccessSecurityMatrixTest
docker compose exec app php artisan test --filter=HoldingAccessHttpSecurityMatrixTest
```

---

## ۶. گردش کار استاندارد هر صفحه

```
A. شناسایی صفحه + فایل(های) SoT
B. git log / baseline SHA + خواندن کامل baseline و فعلی
C. جدول delta برای کاربر:
   | مورد | baseline | فعلی | اقدام |
D. فقط diff لازم (restore از baseline یا minimal patch)
E. عدم دست زدن به صفحات دیگر
F. Self-check چک‌لیست §۴
G. Push develop + دستور fetch/pull/log/diff برای کاربر
H. درخواست تأیید بصری کاربر فقط برای همان صفحه
```

**ممنوع:** یک commit که ده‌ها صفحه را با هم «درست» کند بدون گزارش جدا.

---

## ۷. گزارش به کاربر (قالب اجباری)

برای هر صفحه:

```markdown
### صفحه: <نام>
- Baseline SHA: <hash> — دلیل انتخاب
- فایل‌های ویرایش‌شده: ...
- امکانات برگردانده‌شده: ...
- عمداً نگه داشته‌شده (امنیت/منطق): ...
- چک‌لیست UI: پاس / مورد باز
- تست BE مرتبط: اجرا شد / ن/م
```

---

## ۸. ترتیب پیشنهادی بازیابی (اولویت بصری کاربر)

1. Members list + helpers + impl  
2. Member detail + AssignRolesCard + AssignScopesCard  
3. Roles list + role detail  
4. Scopes list  
5. Role-assignment-requests  
6. Org companies list + company detail  
7. Branches list + departments list  
8. بقیهٔ Identity advanced (cert / privileged / SoD) در صورت شکایت صریح  

---

## ۹. جملهٔ فرمان برای شروع کار (کپی به چت)

> طبق `UI_Restoration_Playbook_Identity_Org_v1.0.md` صفحهٔ «…» را با baseline git بازیابی کن؛ فقط همان صفحه؛ لیست delta بده؛ امنیت ADR-003/005/006 نشکند؛ بعد push روی develop.

---

**LOCKED 2026-10-19 — agent must follow this playbook for Identity/Org UI restoration.**
