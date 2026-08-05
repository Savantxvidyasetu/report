# VidyaSetu OEMS — Microservices Status Report (Stakeholder)

**Date:** 2026-08-05  
**Source of truth:** Repo-wise code mapping (19 repos) + live dev host `3.110.54.199`  
**Audience:** Stakeholders / leadership  

---

## Executive summary

| Metric | Count | Notes |
|--------|------:|-------|
| **Repos in migration map** | **19** | 12 domain + 3 BFF + 4 platform/shared |
| **Domain microservices live on dev** | **7 / 12** | Healthy Docker containers |
| **Domain microservices still pending** | **5 / 12** | Not extracted / not deployed |
| **Separate BFF repos (per map)** | **0 / 3** | Temporary single Laravel BFF instead |
| **React frontend (`vidyasetu-web`)** | **Not started** | UI still Blade in `vidyasetu-frontend` |
| **Platform / contracts repos** | **Not started** | Helm/Kong/OpenAPI TBD |
| **Monolith (`oems-main`)** | **Still running** | Strangler; remaining domains still live here |

**Bottom line for stakeholders:** Core tenant + exam stack is **live on the shared development host**. Roughly **~58% of domain services** are extracted and running. **5 domain services**, the **React portal**, **dedicated BFFs**, and **platform/contracts** work remain. System still uses a **shared MySQL database** (not yet schema-per-service) and **Blade BFF** (not React).

---

## Overall project status

| Area | Status | Stakeholder message |
|------|--------|---------------------|
| **Identity & access** | ✅ Done (dev) | Login/OTP/JWT for student, admin, superadmin |
| **School / tenant** | ✅ Done (dev) | Schools, settings, staff onboarding requests |
| **Exam engine** | ✅ Done (dev) | Exam CRUD, slots, attempts, results API |
| **Proctoring** | ✅ Done (dev) | Violations, monitor blocks, LiveKit |
| **Media / S3** | ✅ Done (dev) | Presigned uploads/downloads |
| **Student domain** | ✅ Done (dev) | Students, profiles, e-learning API; BFF wired |
| **Admin domain** | ✅ Done (dev) | Admins, staff, requests, roles API; BFF wired |
| **Questions** | ❌ Pending | Still in monolith |
| **Notifications** | ❌ Pending | Still in monolith / partial |
| **Settings (global)** | ❌ Pending | Still in monolith |
| **Reporting / audit** | ❌ Pending | Still in monolith |
| **Registration / payments** | ❌ Pending | Olympiad + gateways still in monolith |
| **React web + 3 BFFs** | ❌ Pending | Map target; interim Laravel Blade BFF |
| **DevOps platform / contracts** | ❌ Pending | Not built as map repos yet |
| **Production readiness** | ⚠️ Partial | Shared DB, shared Redis, no Kafka yet, some Eloquent fallbacks |

**Phase (vs migration order in mapping doc):** Through **order 4** (school / student / admin) on the **dev host**. Order **5** domains and **parallel** web/BFF/contracts not started.

---

## 1. Domain microservices (12) — completed vs pending

### ✅ Completed & deployed on development (`3.110.54.199`)

| # | Repo / service | Port | Status | Smoke / notes |
|---|----------------|-----:|--------|---------------|
| 1 | **vidyasetu-identity** | 3001 | Healthy | Login + OTP + JWT; BFF uses it |
| 2 | **vidyasetu-school** | 3002 | Healthy | School CRUD, staff requests; BFF wired |
| 3 | **vidyasetu-exam-engine** | 3003 | Healthy | Exam APIs; BFF ExamServiceClient |
| 4 | **vidyasetu-proctoring** | 3004 | Healthy | LiveKit + violations; BFF wired |
| 5 | **vidyasetu-media** | 3007 | Healthy | Presigned URLs; JWT multi-actor fix |
| 6 | **vidyasetu-student** | 3012 | Healthy | **43/43** API curl PASS (2026-08-02); BFF client |
| 7 | **vidyasetu-admin** | 3005 | Healthy | **24/24** API curl PASS (2026-08-02); BFF client |

**Domain completed: 7**  
**Domain pending: 5**

### ❌ Pending domain microservices (not deployed / not extracted)

| # | Repo / service | Owns (from map) | Current home | Blocking / risk |
|---|----------------|-----------------|--------------|-----------------|
| 8 | **vidyasetu-question-bank** | Questions, passages, import | Monolith | Exam-engine still needs questions from shared DB / mono |
| 9 | **vidyasetu-notification** | Email, SMS, in-app notify | Monolith / partial | OTP email may still be local; no Kafka fan-out |
| 10 | **vidyasetu-settings** | Global settings, exam rule defaults, flags | Monolith | Admin role-matrix still dual-homed transitional |
| 11 | **vidyasetu-reporting** | Analytics, security logs, exports, PDF | Monolith | No central audit bus yet |
| 12 | **vidyasetu-registration** | Olympiad registration, payments, slots, refunds | Monolith | Highest risk domain (PCI / webhooks) |

---

## 2. BFF repos (3) — map vs reality

| # | Map target | Status | Reality today |
|---|------------|--------|---------------|
| 13 | vidyasetu-bff-student | ❌ Not created | Combined into **vidyasetu-frontend** (Laravel Blade BFF) |
| 14 | vidyasetu-bff-admin | ❌ Not created | Same |
| 15 | vidyasetu-bff-superadmin | ❌ Not created | Same |

**Interim BFF (`vidyasetu-frontend` :3000):** ✅ Deployed healthy. Clients live for identity, school, exam, proctoring, media, student, **admin**. Pattern: call microservice first, Eloquent fallback if down.

**Gap for stakeholders:** Map calls for **3 API BFFs + React**. We have **1 temporary Blade BFF**. Acceptable for strangler phase; not the end-state architecture.

---

## 3. Platform & shared (4)

| # | Repo | Status | Notes |
|---|------|--------|-------|
| 16 | **vidyasetu-web** (React) | ❌ Pending | All UI still Blade |
| 17 | **vidyasetu-platform** (Helm/TF/Kong) | ❌ Pending | Dev uses Docker Compose on one host |
| 18 | **vidyasetu-contracts** (OpenAPI/Kafka schemas) | ❌ Pending | No shared contracts repo yet |
| 19 | **oems-main** (monolith) | ⚠️ Still active | Deployed `:3010`; remaining domains + legacy routes |

---

## 4. Live infrastructure (dev) — as of 2026-08-05

| Component | Status |
|-----------|--------|
| MySQL + Redis + Consul | Running |
| LiveKit SFU | Running (`:7880`) |
| Microservices above (7) + frontend + monolith | Healthy |
| Kafka / MSK | **Not in use** (map assumes async events) |
| Schema-per-service DBs | **Not done** (shared `vidyasetu` DB) |
| Keycloak | **Not done** (identity RS256 JWT today) |
| Gateway (Kong) blocking browser→micro | **Not done** (BFF pattern in place) |

---

## 5. What “done” means vs what remains (honest caveats)

**Done on dev means:** service repo exists, container healthy, JWT + internal key, core APIs smoke-tested, BFF can call for extracted screens.

**Not done yet (even for completed services):**
- Full cutover (some Blade controllers still have Eloquent fallback)
- Schema isolation / no cross-service DB joins
- Kafka event bus + reporting ingest
- Production secrets manager / mTLS
- React UI replacement
- Admin gaps called out in review: identity disable API, sub-SA section gates, role-matrix ownership vs school/settings

---

## 6. Suggested talking points for the meeting

1. **Progress:** 7 of 12 domain microservices are **live on the development environment**, covering the migration order through **identity → exam → proctoring/media → school/student/admin**.
2. **Pending domain work:** **5 services** — question-bank, notification, settings, reporting, registration (payments is the highest-risk remaining slice).
3. **Frontend:** Users still use **Blade**. Target React (`vidyasetu-web`) + 3 BFFs are **future**; current Laravel BFF is the bridge.
4. **Monolith:** Still required for olympiad/payments, questions, notifications UI, reports, and any screen not yet on a client.
5. **Ask / next decisions:** Prioritize next extract (**registration** vs **question-bank** vs **notification**), and whether to start **React/web** in parallel.

---

## 7. One-slide scoreboard

```text
DOMAIN MICROSERVICES     ███████░░░░░  7/12 live   |  5 pending
BFF REPOS (map)          ░░░░░░░░░░░░  0/3        |  interim 1 Blade BFF
REACT WEB                ░░░░░░░░░░░░  0/1        |  pending
PLATFORM + CONTRACTS     ░░░░░░░░░░░░  0/2        |  pending
MONOLITH                 ████████░░░░  active     |  strangler continues
```

**Pending microservices (domain only): 5**  
**Pending if counting map BFFs + web + platform + contracts as “not started repos”: 5 + 3 + 3 = 11 of 19** (monolith counts as active temporary, not “pending create”).

---

*Generated from platform memory + live `docker ps` on 2026-08-05. Update after each new service deploy.*
