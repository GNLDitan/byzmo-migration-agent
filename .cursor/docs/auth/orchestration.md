# Auth Execution + Frontend Orchestrator

How to run **storefront Auth / Login** on `byzmo-v2/web` using clean-architecture layers (same pattern as Catalog / Cart).

Related: [`checklist.md`](./checklist.md) · [`../migration-progress.md`](../migration-progress.md) · [`../module-inventory.md`](../module-inventory.md) · [`../cart/orchestration.md`](../cart/orchestration.md)

---

## Scope

Legacy has **no dedicated `/login` page** — login is a **modal** (nav, also triggered from cart/PDP/shipping). This module covers:

- Email/password login → `POST /user/authenticate`
- Session restore via `token` + `email` LS → `POST /user/getuserbyemail`
- Logout (clear token/email; clear cart LS)
- Cart merge on login (`PATCH /cart/updatecartuser`) — already in Cart presenters
- Login UI (modal + optional `/login` entry) for storefront

**Deferred to later modules / waves**

- Signup / forgot password → **Account** module (`/signup`, `/forgot/*`)
- Guest payment token (`/user/loginguesttoken`, `/payment/auth/login-guest/:id`) → **Payments**
- Google / Facebook social → Auth Wave 4+ (optional) after email/password works
- Boilerplate NextAuth (`domain/Application/Authentication`, `/login` Clean template) stays for admin/demo — **do not reuse for Byzmo storefront contracts**

---

## Prerequisites

1. Catalog + Cart done
2. Byzmo API reachable (`API_URL`)
3. Preserve `/user/authenticate` + `/user/getuserbyemail` contracts
4. One wave at a time

---

## Architecture map (Auth)

| Layer | Path | Auth work |
|-------|------|-----------|
| **Domain** | `src/domain/Auth/` | `User`, session/auth result models; repo interfaces; login / fetch-by-email use cases |
| **Data** | `src/data/network/rest/v1/Auth/` | REST against `/user/...`; DTOs; mappers |
| **IoC** | `src/ioc/Auth/` | Bind Auth repos / use cases / presenters (separate from `AuthenticationModule`) |
| **Presentation** | `src/presentation/Auth/` | Login presenter; session view models |
| **UI** | `src/ui/Auth` (+ header Account) | Login modal; wire CartSession to real `userId` / `isAdmin` |
| **Client session** | LS + context | Legacy keys: `token`, `email` (+ existing `guesttoken`, `browsertoken`) |

**Flow:** UI → Presenter → Use case → Repository → REST → API

---

## Critical business rules (preserve)

1. Authenticate body: `{ email, password }` (or social: `{ email, isSocialMediaLogin: true, name }`)
2. Success stores LS `token` + `email` (not user id)
3. User id / `isAdmin` / `name` loaded via `getuserbyemail` (requires Bearer token)
4. Admin login → legacy navigates to `/administration` (v2: keep admin out of storefront cart mutate)
5. Customer login → merge guest cart by IP then reload carts
6. Logout removes `token`, `email`, and cart LS key `cart`
7. Interceptor prefers `guesttoken` over `token` when both set (payment guest)

### API surface (this module)

| Method | Path | Notes |
|--------|------|-------|
| POST | `/user/authenticate` | AllowAnonymous; returns `{ name, email, isAdmin, isActive, token }` |
| POST | `/user/getuserbyemail` | Auth required; body `{ email }` → full `User` + addresses |

Cart merge (already Cart): `PATCH /cart/updatecartuser` with `{ id, ipAddress }`

---

## Waves

### Wave 0 — Foundation
- This playbook + checklist
- Byzmo session LS helpers (`token`, `email` get/set/clear)
- Storefront session context (or extend cart session) to expose `userId` / `isAdmin` / `email` / `ready`
- Header Account link ready for login modal (placeholder OK)
- Update `migration-progress.md` (Auth in-progress)

### Wave 1 — Domain
- Models + `IAuthRepository` + use cases (authenticate, fetch user by email)
- No React, no URLs in domain
- Do **not** conflate with `domain/Application/Authentication`

### Wave 2 — Data + IoC
- REST clients + mappers + `AuthModule` in container

### Wave 3 — Presentation
- Login presenter (authenticate → persist session → fetch user → optional cart merge handoff)
- Logout / restore-session presenter APIs

### Wave 4 — UI + App
- Login modal (match Catalog visual language / nav)
- Wire header Account; cart checkout `requiresLogin` opens modal
- On login success: call Cart `mergeOnLogin` + refresh badge
- Keep `/login` as thin entry that opens same flow (legacy had no page)

### Wave 5 — Verify
- Live authenticate + getuserbyemail
- Guest cart merges after login
- Logout clears session + cart count
- Mark Auth done in progress doc

**Do not** implement full Account (signup/profile) or Payments guest inside Auth waves.

---

## Copy-paste orchestrator prompt

```text
Follow AGENTS.md, .cursor/docs/auth/orchestration.md, .cursor/docs/auth/checklist.md.
Use clean architecture in byzmo-v2/web (same patterns as Catalog/Cart).
New bounded context domain/Auth — do not reuse boilerplate Application/Authentication for Byzmo APIs.
Preserve /user/authenticate and /user/getuserbyemail contracts.
Execute ONLY Wave N: <name>. Do not implement later waves.
Update checklist and .cursor/docs/migration-progress.md when done.
```

---

## Done when

Storefront users can log in (email/password), session restores on reload, cart merges on login, logout works, checkout login gate opens login UI, progress doc updated.
