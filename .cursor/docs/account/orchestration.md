# Account Execution + Frontend Orchestrator

How to run **storefront Account** (signup, forgot password, profile, addresses) on `byzmo-v2/web` using clean-architecture layers (same pattern as Catalog / Cart / Auth / Checkout).

Related: [`checklist.md`](./checklist.md) · [`../migration-progress.md`](../migration-progress.md) · [`../module-inventory.md`](../module-inventory.md) · [`../auth/orchestration.md`](../auth/orchestration.md) · [`../checkout/orchestration.md`](../checkout/orchestration.md)

---

## Scope

| Surface | URL | Legacy component |
|---------|-----|------------------|
| Sign up | `/signup` | Signup |
| Forgot password | `/forgot` | ForgotPassword |
| Email sent | `/forgot/email-sent` | ForgotPasswordEmailSent |
| Reset password | `/forgot/verify?email=&token=` | ForgotPasswordVerify |
| Profile home | `/profile` → `/profile/profile-dashboard` | ProfileDashboard |
| Information | `/profile/profile-dashboard/information` | Information |
| Addresses | `/profile/profile-dashboard/addresses` (also nested under information) | Addresses |

This module covers:

- Customer registration (`createuser` + email uniqueness check)
- Forgot / reset password (`sendresetpasswordlink` → `istokenvalid` → `updateuserpassword`)
- Profile information edit (`updateuser`, optional password change)
- Address book CRUD (`createuseraddress` / `updateuseraddress` / `deleteuseraddress` / list by userId)
- Auth-gated profile area (reuse Auth session; redirect / open login when anonymous)

**Reuse from Auth**

- Session LS (`token` / `email`), `getuserbyemail`, login modal, `User` model
- Do **not** reimplement authenticate inside Account

**Reuse / extend from Checkout**

- Checkout already **reads** addresses (`getuseraddressbyuserid`) for shipping pickers
- Account owns full address CRUD; later waves may share mappers/models (prefer `domain/Account` + thin Checkout adapter, or move shared address types)

**Deferred**

- Payments guest login (`/payment/auth/login-guest/:id`) → Payments
- Wishlist / product notifications / email subscribe → Loyalty / extras
- Customer order history (`/client-order`) → Customer orders module
- Admin user CRUD → Admin
- Social signup (Google / Facebook) — optional after email signup works

**Payments** remains deferred (gateway UIs skipped for later).

---

## Prerequisites

1. Catalog + Cart + Auth + Checkout done
2. Byzmo API reachable (`API_URL`)
3. Preserve `/user/*`, `/email/sendresetpasswordlink`, `/resettoken/istokenvalid` contracts
4. One wave at a time

---

## Architecture map (Account)

| Layer | Path | Account work |
|-------|------|--------------|
| **Domain** | `src/domain/Account/` | Profile / address / reset-token models; validators; repo interfaces; use cases |
| **Data** | `src/data/network/rest/v1/Account/` | REST DTOs, mappers, repos |
| **IoC** | `src/ioc/Account/` | Bind repos / use cases / presenters |
| **Presentation** | `src/presentation/Account/` | Signup / forgot / profile / addresses presenters + VMs |
| **UI** | `src/ui/Account` | Forms + profile shell |
| **Session** | Auth client session | Gate profile routes; post-signup auto-login |

**Flow:** UI → Presenter → Use case → Repository → REST → API

---

## Critical business rules (preserve)

1. Signup fields: name, email, mobileNumber, password, confirmPassword — password regex ≥8 with letter + digit (legacy `passwordValidator`)
2. Debounced `checkemailvalidity` before / on submit; block duplicate emails
3. Logged-in users hitting `/signup` → home (legacy)
4. After successful signup → typically login / home (preserve legacy toaster + navigate home)
5. Forgot: POST email → navigate email-sent; verify page reads `email` + `token` query params
6. Reset: `updateuserpassword` with `email` + `resetToken` + new password → then authenticate
7. Profile / addresses require LS token; invalid session → home / login
8. Addresses: default flag; PH vs intl region fields (reuse Checkout region helpers where possible)
9. Information: update profile; password change only when password fields filled

### API surface (this module)

| Method | Path | Notes |
|--------|------|-------|
| POST | `/user/createuser` | Register |
| POST | `/user/checkemailvalidity` | Email uniqueness |
| PATCH | `/user/updateuser` | Profile |
| PATCH | `/user/updateuserpassword` | Password / reset |
| POST | `/email/sendresetpasswordlink` | Body `{ email }` |
| POST | `/resettoken/istokenvalid` | Optional guard for verify page |
| POST | `/user/createuseraddress` | Address create |
| PATCH | `/user/updateuseraddress` | Address update |
| POST | `/user/deleteuseraddress` | Address delete |
| GET | `/user/getuseraddressbyuserid/{id}` | List (also Checkout) |
| POST | `/user/getuserbyemail` | Hydrate profile (Auth) |

---

## Waves

### Wave 0 — Foundation
- This playbook + checklist
- Storefront routes + placeholder pages for signup / forgot* / profile*
- `AccountEndpoints` consts
- Header: logged-in Account → profile
- Update `migration-progress.md` (Account in-progress; Payments stays deferred)

### Wave 1 — Domain
- Models + validators + repo interfaces + use cases (no React / URLs)

### Wave 2 — Data + IoC
- REST clients + mappers + `AccountModule`

### Wave 3 — Presentation
- Signup / forgot / profile / addresses presenters + VMs

### Wave 4 — UI + App
- Real forms + profile shell; wire Auth session gates

### Wave 5 — Verify
- Live signup, reset smoke, profile update, address CRUD
- Mark Account done; Payments still deferred

---

## Copy-paste orchestrator prompt

```text
Follow AGENTS.md, .cursor/docs/account/orchestration.md, .cursor/docs/account/checklist.md.
Use clean architecture in byzmo-v2/web (same patterns as Catalog/Cart/Auth/Checkout).
New bounded context domain/Account — reuse Auth session; do not reimplement login.
Preserve /user create/update/address and reset-password contracts.
Execute ONLY Wave N: <name>. Do not implement later waves.
Update checklist and .cursor/docs/migration-progress.md when done.
```

---

## Done when

Guests can sign up and reset password; logged-in users can edit profile and manage addresses; Auth login/signup links land on these pages; progress doc updated; Payments still out of scope.
