# Holding Access Security Scenario Matrix v1.0

| Field | Value |
|-------|--------|
| **Status** | ACTIVE |
| **Date** | 2026-10-18 |
| **Laws** | ADR-ID-ORG-003, 004, 005, 006 |
| **Automation** | `HoldingAccessSecurityMatrixTest` → `storage/app/security_matrix/holding_access_latest.json` |

## Result vocabulary

| Result | Meaning |
|--------|---------|
| `PASS` | رفتار با قانون یکی است |
| `FAIL` | رفتار خلاف قانون (باگ امنیتی) |
| `GAP` | قانون روشن است ولی enforcement ناقص/ندارد |
| `SKIP` | پیش‌نیاز fixture/سیدر موجود نبود |

| Severity | Meaning |
|----------|---------|
| `critical` | privilege expansion بین شرکت‌ها / نقش tenant |
| `high` | manage/sibling یا سقف نقش |
| `medium` | consistency |
| `low` | مستندسازی |

## A. Conventional

| ID | Scenario | Expect |
|----|----------|--------|
| HSM-C01 | لیست اعضا فقط شرکت خود | only same company |
| HSM-C02 | عدم manage عضو sibling | DENY |
| HSM-C03 | اعطای نقش company به CHILD توسط owner | ALLOW |
| HSM-C04 | مالک → platform-admin روی is_owner | ALLOW (manual/UI) |
| HSM-C05 | company-op نقش company در سقف خود | ALLOW (manual/UI) |

## B. Complex

| ID | Scenario | Expect |
|----|----------|--------|
| HSM-X01 | platform-admin به کاربر فقط CHILD | DENY |
| HSM-X02 | group-observer به کاربر فقط CHILD | DENY |
| HSM-X03 | اسکوپ primary ⇒ ظرفیت ≥ group | rank≥3 |
| HSM-X04 | بازیگر company نتواند نقش group بدهد | DENY |
| HSM-X05 | grant اسکوپ peer | DENY |
| HSM-X06 | home خارج سقف | DENY (covered by ceiling tests) |
| HSM-X07 | subtree primary: child in / peer out | pass |

## C. Adversarial

| ID | Scenario | Expect |
|----|----------|--------|
| HSM-A01 | multi-company + اسکوپ خالی ≠ group-wide | not group-wide |
| HSM-A02 | خودارتقا platform-admin | DENY |
| HSM-A03 | manage sibling user | DENY |
| HSM-A04 | observer assign | DENY (manual) |
| HSM-A05 | نقش soft-deleted در list | hidden |
| HSM-A06 | replay grant after soft-delete | DENY (manual) |

## Automation

```bash
docker compose exec app php artisan test --filter=HoldingAccessSecurityMatrixTest
docker compose exec app cat storage/app/security_matrix/holding_access_latest.json
```

## Manual demo seeder

```bash
docker compose exec app php artisan db:seed --class=Database\\Seeders\\HoldingPlatformTestSeeder
```

Accounts: `owner@demo.local`, `observer@demo.local`, `op.a@demo.local`, `op.b@demo.local`.
