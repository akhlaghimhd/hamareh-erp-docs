# SAASADM Layer 2 — Final status (2026-10-06)

## Result

**Layer 2 SaaS Admin track (planned P0–P5 BE + P7 FE) is CLOSED.**

| Wave | Repo | Status |
|------|------|--------|
| P0 Feature catalog + entitlement | hamarehSaasErp | CLOSED |
| P1 Org FeaturePackGate | hamarehSaasErp | CLOSED |
| P2 Admin IAM | hamarehSaasErp | CLOSED |
| P3 System settings + retention defaults | hamarehSaasErp | CLOSED |
| P4 Audit (login / GRANT / REVOKE) | hamarehSaasErp | CLOSED |
| P5 Admin tenant list/show | hamarehSaasErp | CLOSED |
| P6 Notifications / Support | — | DEFERRED (skeleton only) |
| P7 FE `/admin` shell | hamarehSaasErp-Front | CLOSED |
| Dual-approval tenant settings formalization | hamarehSaasErp-Front | CLOSED |

## Follow-on (post Layer 2)

| Item | Status |
|------|--------|
| `erp:purge-soft-deleted` Org masters P0 | **SHIPPED** BE `53a6b8cc` (gated by `retention.purge_job_enabled`) |
| Tenant retention UX on deleted buckets | OPEN (DEBT-ORG-007 residual FE) |
| Identity list purge types | OPEN (DEBT-ID-003) |

## FE routes

- `/admin/login` — platform admin auth  
- `/admin/tenants` — tenant list  
- `/admin/feature-packs` — grant/revoke (supports `?tenantId=`)  
- `/admin/system-settings` — platform keys (retention.*)  
- `/dashboard/identity/settings` — **tenant** dual-approval SoT  

## Debts closed

- DEBT-PLT-001, DEBT-PLT-002  
- DEBT-SAAS-001, DEBT-SAAS-002  

## Still open (not Layer 2 blockers)

- DEBT-PLT-003 residual — tenant retention UX  
- DEBT-ORG-007 residual FE — deleted-bucket messaging  
- DEBT-ID-003 — Identity purge entities  
- DEBT-ID-009 Holding H4/H5 product rules  
- Optional P6 support/notifications  

## Local admin seed (APP_ENV=local)

`platform.admin` / `LocalAdmin1!` (override with `PLATFORM_ADMIN_*` env in production)

## Purge command

```bash
docker compose exec app php artisan erp:purge-soft-deleted
# override gate:
docker compose exec app php artisan erp:purge-soft-deleted --force
```

Scheduled daily 03:30; no-op unless `retention.purge_job_enabled=true` in system_settings.
