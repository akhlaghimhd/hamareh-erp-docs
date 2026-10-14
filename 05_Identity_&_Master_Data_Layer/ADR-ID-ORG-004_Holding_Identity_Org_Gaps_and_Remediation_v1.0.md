# ADR-ID-ORG-004 — Holding Identity & Organization Gaps and Remediation

| Field | Value |
|-------|--------|
| **Status** | **ACCEPTED** (product direction locked 2026-10-14) |
| **Date** | 2026-10-14 |
| **Amends** | ADR-ID-ORG-003 (does not replace single-tenant holding model) |
| **Aligns with** | ADR-FIN-003 (FIN-HOLD-L01…L16) |
| **Related debts** | DEBT-ID-H01… / DEBT-ORG-012 / DEBT-FIN-HOLD-* |

---

## 0. Owner intent (locked)

- Holding **observes** (filter/report); does **not** freely operate child books or inflate permissions downward.
- Each **company** (any depth) manages **its own** users within Scope.
- **Role power** must not leak from parent catalog to child operators beyond what Scope + grant rules allow.
- **Branch** admin is a real product need if multi-branch is sold — Identity must eventually understand BRANCH, not only COMPANY.
- **Scope remains the authorization mechanism** (do not replace with org-tree magic). Harden it; do not abandon it.
- Target: mid-size SaaS holdings, not unbounded mega on-prem trees (depth → feature pack / DEBT-ORG-012).

All **25 remediation items** below are **accepted** as the gap list vs current Identity + Organization behaviour.

---

## 1. Gap register (25 items)

### A. Power boundary

| ID | Title | Current | Remediation |
|----|-------|---------|-------------|
| **G01** | Tenant `is_owner` = power everywhere | Group-wide visibility and broad operate paths | Split: technical owner ≠ group observer/operator; child write only with explicit operate grant |
| **G02** | Empty COMPANY scopes soft-open path | SME-friendly; dangerous if multi-company tenant mis-seeded | Multi-company tenant: empty COMPANY scope ≠ group access unless explicit observer role |
| **G03** | Group read ≠ child write | Locked in ADR-FIN-003; Identity not fully aligned | Same split on Identity admin and all ops modules |
| **G04** | Permission inflation via shared role catalog | Roles/permissions tenant-wide; child with `identity.role.*` reaches full catalog | Cap who creates roles; cap assignable permission sets per layer/Scope |
| **G05** | Holding more roles than child / branch less | No layered catalog | Role templates or assignable-permission ceilings per layer (even if one physical role table) |

### B. User & role management per layer

| ID | Title | Current | Remediation |
|----|-------|---------|-------------|
| **G06** | Child company admin → own members only | Largely OK via COMPANY Scope on member APIs | Keep; add depth (grandchild) tests; close bypass paths |
| **G07** | Role assign only inside Scope | Enforced on assign-to-user | Keep; add “roles allowed for this company” filter |
| **G08** | Role/permission CRUD boundary | No company boundary on createRole | Local vs group roles, or createRole restricted to group security admin |
| **G09** | Branch manager → branch users/roles | HoldingAccess is COMPANY-centric | BRANCH Scope on member admin **or** explicit product decision that branch is not an Identity admin unit |
| **G10** | Grant Scope only within actor’s scopes | Partially intended | Harden + tests: actor cannot grant Scope they do not hold |

### C. Org tree depth

| ID | Title | Current | Remediation |
|----|-------|---------|-------------|
| **G11** | Multi-level company tree | Supported in Org model | Never auto-grant **operate** on all descendants via legal parent edge |
| **G12** | Scope does not inherit from legal tree | Intentional (anti-leak) | Group observer gets explicit subtree/read grants — not silent hierarchy auth |
| **G13** | Depth caps | Not in feature pack | **DEBT-ORG-012** Platform Owner / pack limits |
| **G14** | Nested branch vs Identity | Branch model exists; Identity weak | If branch is a management unit, Identity must evaluate BRANCH |

### D. Seed / defaults / demo

| ID | Title | Current | Remediation |
|----|-------|---------|-------------|
| **G15** | Flat roles under parent in demo/seed | Likely simple-tenant oriented | Seed: group-observer, company-operator, read-only group templates |
| **G16** | Owner gets all active permissions on login | Historical pattern | Narrow owner pack; align with G01 |
| **G17** | Permission catalog lacking layer tags | Flat permission list | Tag/group: Group-Read / Company-Operate / Platform-Admin |

### E. Organization structure

| ID | Title | Current | Remediation |
|----|-------|---------|-------------|
| **G18** | Child company create / single root | Rules exist | Keep; bind to depth pack (G13) |
| **G19** | Feature packs multi_company / branch / BU | Independent — correct | Ensure all create/list surfaces remain gated |
| **G20** | Legal hierarchy ≠ authorization | ADR-ID-ORG-003 — correct | Keep; reports may walk tree; auth stays Scope |
| **G21** | Intercompany Org P1 | Partners/rules closed; rest parked | Stay aligned with FIN-HOLD-L14 |

### F. Cross-module alignment

| ID | Title | Current | Remediation |
|----|-------|---------|-------------|
| **G22** | Same access law all ops modules | FIN locked; ID/Org lag | Finish Identity harden before next ops module expansion |
| **G23** | Dual role-assignment approval | Separate path | Do not conflate with group observer; keep for sensitive grants |
| **G24** | Identity audit trail | Weaker than FIN-HOLD-L15 intent | Audit role/Scope grant and access denials |
| **G25** | FE members holding UX | Partial company filter | Observer read-only; company admin only own company |

---

## 2. Priority bands

| Band | Items | Meaning |
|------|-------|--------|
| **P0** | G01 G02 G03 G04 | Power boundary — without these, rest is cosmetic |
| **P1** | G05 G15 G16 G17 | Observer role + seed/catalog shape |
| **P2** | G06 G07 G08 G10 | Child company admin completeness |
| **P3** | G09 G14 | Branch as Identity admin unit |
| **P4** | G11 G12 G13 G18–G25 | Depth, packs, IC, audit, FE, cross-module |

---

## 3. Non-negotiables

1. Do **not** replace Scope with org-tree-only authorization.
2. Do **not** give parent **operate** on children by default.
3. Do **not** leave tenant-wide role CRUD unrestricted for every company admin.
4. Single tenant per holding remains (ADR-ID-ORG-003).

---

## 4. Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-10-14 | 25 gaps accepted; work breakdown companion doc |
