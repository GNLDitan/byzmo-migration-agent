# Migration Progress

Target: `byzmo-v2/web` (Next.js/React template). Backend API unchanged.

Doc index: [`README.md`](./README.md) · workflow: [`workflow.md`](./workflow.md) · per-module playbooks under `.cursor/docs/<module>/`

**Status key:** `todo` · `in-progress` · `done` · `blocked`

## Current focus

| Field | Value |
|-------|--------|
| Active module | Customer orders |
| Status | in-progress |
| Notes | Wave 1 domain done. Payments still deferred. After Orders: Blog/static, Loyalty. |

## Foundation

| Item | Status | Notes |
|------|--------|-------|
| Scaffold `byzmo-v2/web` | done | Renamed from `clean-architecture-nextjs-react-boilerplate` |
| Environments + API base URL | done | `API_URL` → Azure `:5549/api`; image CDN still `api.shopbyzmo.com` |
| HTTP interceptor + auth headers | done | RestClient baseURL + Byzmo token/guesttoken |
| Shared models / types | done | `domain/Catalog` models + use cases (Wave 1) |
| App shell (nav, footer, layout) | done | `StorefrontLayout` + public `/`, `/products` |

## Storefront

| Module | Status | Notes |
|--------|--------|-------|
| Catalog (home / list / PDP) | done | Live API OK; related uses `categoryFilter`; CDN images may 404 |
| Cart | done | Waves 0–5; guest CRUD + merge verified on Azure |
| Checkout (shipping → method → payment) | done | Waves 0–5; live guest + logged-in place-order; handoff stubs |
| Payments (gateways) | deferred | Skipped for later; Checkout handoff stubs remain |
| Auth / Login (modal + guest + social) | done | Waves 0–5; live auth + merge + logout; social/signup deferred |
| Account (signup / profile / addresses) | done | Waves 0–5; live signup/profile/address; reset SMTP lab-limited |
| Customer orders + reviews | in-progress | Wave 1 — `domain/Orders` models, helpers, use cases |
| Blog + static pages | todo | |
| Loyalty / wishlist / subscribe | todo | |

## Admin

| Module | Status | Notes |
|--------|--------|-------|
| Dashboard | todo | Defer until storefront core works |
| Products / inventory | todo | |
| Categories / landing | todo | |
| Orders | todo | |
| Shipping / payment methods / discounts | todo | |
| Layaway / loyalty admin | todo | |
| Blog admin | todo | |
| User admin | todo | |
| Reports | todo | |

## Decisions / findings

- Legacy app is Angular **8.2**, path `byzmo/web`.
- V2 frontend template: Next.js/React clean-architecture boilerplate at `byzmo-v2/web`.
- Preserve all existing API endpoints and checkout business rules.
- Migrate one module at a time; ask before major architecture changes.
- Cart identity = `{userId}-{ipAddress}` via `browsertoken` (legacy `IpService`); mixing rules differ at add vs checkout.
- Cart Wave 5: live Azure smoke + domain verify script; jest CLI currently blocked by wrap-ansi ESM in this workspace.
- Storefront Auth is `domain/Auth` + `/user/*` — do not reuse boilerplate `domain/Application/Authentication` (NextAuth).
- Auth Wave 5: `npx tsx scripts/auth-wave5-verify.mts` (LS + presenter + live Azure authenticate/merge).
- Account Wave 5: `npx tsx scripts/account-wave5-verify.mts`; Azure SMTP may fail on reset-link; `/resettoken/istokenvalid` 404 on current API build.

## Changelog

| Date | Update |
|------|--------|
| 2026-10-21 | Initial inventory + progress tracker; Catalog recommended first |
| 2026-10-21 | Renamed boilerplate → `byzmo-v2/web`; set as v2 template |
| 2026-10-22 | Added Catalog checklist + orchestration (now `.cursor/docs/catalog/`) |
| 2026-10-22 | Added UI references (now `.cursor/docs/ui/`) |
| 2026-10-22 | Wave 0 complete — env, HTTP auth, storefront shell |
| 2026-10-22 | Wave 1 complete — `domain/Catalog` models, repos, use cases |
| 2026-10-22 | Wave 2 complete — Catalog REST repos, mappers, IoC |
| 2026-10-22 | Wave 3 complete — Home/Products/PDP presenters + view models |
| 2026-10-22 | Wave 4 complete — storefront UI from references + stub cart CTA |
| 2026-10-28 | Wave 5 complete — Catalog acceptance vs Azure API; related filter fix |
| 2026-10-30 | Cart Wave 0–1 — orchestration/checklist, browsertoken bootstrap, `domain/Cart` |
| 2026-10-30 | Cart Wave 2 — `/cart` + layaway REST repos, mappers, `CartModule` IoC |
| 2026-10-30 | Cart Wave 3 — Cart / AddToCart / CartHeader presenters + VMs |
| 2026-10-30 | Cart Wave 4 — `/cart` UI, AddToCartButton, header badge, shipping handoff |
| 2026-10-30 | Cart Wave 5 — Azure CRUD/merge smoke + domain gates; Cart → done |
| 2026-10-30 | Reorganized docs: `.cursor/docs/catalog/`, `.cursor/docs/cart/` (+ index) |
| 2026-10-30 | Moved migration docs to `.cursor/docs/` |
| 2026-10-30 | Auth Wave 0–1 — docs, session helpers, `domain/Auth`, CartSession wired to user |
| 2026-10-30 | Auth Wave 2 — `/user/authenticate` + `getuserbyemail` REST, `AuthModule` IoC |
| 2026-10-30 | Auth Wave 3 — LoginPresenter (login / restore / logout) + VMs |
| 2026-10-30 | Auth Wave 4 — login modal, session restore, cart merge, logout |
| 2026-10-30 | Auth Wave 5 — Azure auth/merge smoke + LS/presenter verify; Auth → done |
| 2026-10-30 | Checkout Wave 0 — docs, 3-step routes, CheckoutSession, shell placeholders |
| 2026-10-30 | Checkout Wave 1 — domain models, totals helpers, place-order use case |
| 2026-10-30 | Checkout Wave 2 — shipping/order/payment REST repos + CheckoutModule IoC |
| 2026-10-30 | Checkout Wave 3 — shell/details/method/payment presenters + VMs |
| 2026-10-30 | Checkout Wave 4 — shipping/method/payment UI, place order, handoff stubs |
| 2026-10-30 | Checkout Wave 5 — Azure place-order smoke + gates; Checkout → done |
| 2026-10-30 | Payments deferred; Account Wave 0 — docs, routes, placeholders, endpoints |
| 2026-10-30 | Account Wave 1 — domain models, validators, use cases |
| 2026-10-30 | Account Wave 2 — REST repos + AccountModule IoC |
| 2026-10-30 | Account Wave 3 — signup/forgot/profile/addresses presenters |
| 2026-10-30 | Account Wave 4 — signup/forgot/profile/addresses UI + auth gate |
| 2026-10-30 | Account Wave 5 — Azure signup/profile/address smoke; Account → done |
| 2026-10-30 | Orders Wave 0 — orchestration/checklist, `/client-order*` placeholders, `OrdersEndpoints`, My Orders nav |
| 2026-10-30 | Orders Wave 1 — `domain/Orders` models, gates, repos, use cases + verify script |
