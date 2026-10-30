# Auth Migration Checklist

Related: [`orchestration.md`](./orchestration.md) · [`../migration-progress.md`](../migration-progress.md) · [`../module-inventory.md`](../module-inventory.md)

**Status key:** `[ ]` todo · `[x]` done

## 0. Foundation (Wave 0)
- [x] Auth orchestration + this checklist
- [x] Session LS helpers (`token`, `email` get/set/clear)
- [x] Storefront session exposes `userId` / `isAdmin` / `email` (CartSession wired)
- [x] Header Account entry ready for login modal (+ local logout)
- [x] Progress: Auth in-progress

## 1. Domain (Wave 1)
- [x] `User` + auth result models
- [x] `IAuthRepository` (authenticate, getUserByEmail)
- [x] Use cases: login authenticate + fetch user by email
- [x] Separate from `domain/Application/Authentication`

## 2. Data + IoC (Wave 2)
- [x] REST endpoints, DTOs, mappers, repos
- [x] `AuthModule` + symbols in container

## 3. Presentation (Wave 3)
- [x] Login presenter (authenticate → user hydrate → cart merge handoff flag)
- [x] Logout / restore session presenter APIs
- [x] `getUserByEmail` accepts bearerToken for server actions

## 4. UI + App (Wave 4)
- [x] Login modal UI
- [x] Header Account + cart `requiresLogin` opens modal
- [x] Login merges cart; logout clears session/cart LS
- [x] Social login deferred (or stub)

## 5. Acceptance (Wave 5)
- [x] Live authenticate + getuserbyemail
- [x] Guest cart merges after login
- [x] Logout clears token/email/cart
- [x] No signup/profile/payment-guest in this slice
- [x] `migration-progress.md` Auth → done

### Wave 5 evidence (2026-10-30, Azure `:5549/api`)
- Script: `npx tsx scripts/auth-wave5-verify.mts` — all assertions passed
- Local: LS logout clears `token`/`email`/`cart`; LoginPresenter login/restore/logout contracts
- Live: ephemeral createuser → authenticate → getuserbyemail (Bearer) → guest cart → `PATCH /cart/updatecartuser` → guest empty / user has lines → deletecart cleanup
- getuserbyemail without Bearer → 401
- Scope: no signup/profile/payment-guest UI in Auth slice (social stubs only)
