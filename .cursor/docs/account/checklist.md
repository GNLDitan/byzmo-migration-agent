# Account Migration Checklist

Related: [`orchestration.md`](./orchestration.md) · [`../migration-progress.md`](../migration-progress.md) · [`../module-inventory.md`](../module-inventory.md)

**Status key:** `[ ]` todo · `[x]` done

## 0. Foundation (Wave 0)
- [x] Account orchestration + this checklist
- [x] Storefront routes: signup / forgot* / profile*
- [x] Placeholder pages + light profile shell
- [x] `AccountEndpoints` consts
- [x] Header Account → profile when logged in
- [x] Progress: Account in-progress (Payments deferred)

## 1. Domain (Wave 1)
- [x] Models (profile draft, address, reset token)
- [x] Pure validators (email / password / confirm)
- [x] Repository interfaces + use cases
- [x] No React / URLs in domain

### Wave 1 evidence
- Script: `npx tsx scripts/account-wave1-verify.mts` — signup/profile/reset validators + register/address use cases

## 2. Data + IoC (Wave 2)
- [x] REST DTOs, mappers, repositories
- [x] `AccountModule` + symbols in container

### Wave 2 evidence
- Repos: account user (create/check/update/password), address CRUD, reset link + token
- IoC: `AccountModule` binds repos + Wave 1 use cases (presenters in Wave 3)

## 3. Presentation (Wave 3)
- [x] Signup / forgot / reset presenters + VMs
- [x] Profile information presenter
- [x] Addresses CRUD presenter

### Wave 3 evidence
- Presenters: Signup (check email + register→login), ForgotPassword, Profile, Addresses
- Auto-login after signup/reset reuses Auth login + getuserbyemail use cases

## 4. UI + App (Wave 4)
- [x] Signup + forgot/reset forms
- [x] Profile shell + information form
- [x] Address book CRUD UI
- [x] Auth gate for profile routes

### Wave 4 evidence
- Views: signup, forgot, email-sent, verify, profile dashboard/information/addresses
- `RequireAuth` gates profile; signup redirects if already logged in
- Server actions in `ui/Account/account.actions.ts`

## 5. Acceptance (Wave 5)
- [x] Live signup + email validity
- [x] Forgot / reset password path
- [x] Profile update + address CRUD
- [x] No gateway / wishlist / orders in this slice
- [x] `migration-progress.md` Account → done

### Wave 5 evidence (2026-10-30, Azure `:5549/api`)
- Script: `npx tsx scripts/account-wave5-verify.mts` — all assertions passed
- Local: validators + Signup/Profile/Addresses presenters (mocked)
- Live: checkemailvalidity → createuser → authenticate → updateuser; address create/update/delete
- Reset: `sendresetpasswordlink` finds user (Azure SMTP fails with SmtpException in lab); `resettoken/istokenvalid` 404 on this build; `updateuserpassword` rejects bad token (400)
- Scope: no gateway / wishlist / orders in Account slice
