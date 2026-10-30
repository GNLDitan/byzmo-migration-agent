# Cart Execution + Frontend Orchestrator

How to run Cart migration on `byzmo-v2/web` **using the clean-architecture layers** (same pattern as Catalog).

Related: [`checklist.md`](./checklist.md) · [`../migration-progress.md`](../migration-progress.md) · [`../module-inventory.md`](../module-inventory.md) · [`../catalog/orchestration.md`](../catalog/orchestration.md)

---

## Prerequisites

1. Catalog done (Home / Products / PDP live)
2. Byzmo API reachable (`API_URL`)
3. Preserve `/api/cart/*` contracts and legacy mixing / login rules
4. Work **one wave at a time**; check off the checklist after each

**Auth note:** Full Auth module can land in parallel or just ahead. Cart Wave 0–3 work with guest `userId = 0` + browser IP. Checkout gates for layaway/pre-order need login (Wave 4 wires merge when Auth session exists).

---

## Architecture map (Cart)

| Layer | Path | Cart work |
|-------|------|-----------|
| **Domain** | `src/domain/Cart/` | Models, repo interfaces, use cases, pure helpers (line totals, mixing, layaway math, checkout gates) |
| **Data** | `src/data/network/rest/v1/Cart/` (+ layaway config via product endpoints) | REST clients, DTOs, mappers → domain; optional LS `'cart'` cache |
| **IoC** | `src/ioc/Cart/` | Bind repos / use cases / presenters |
| **Presentation** | `src/presentation/Cart/` | Cart page + add-to-cart + header count presenters / VMs |
| **UI** | `src/ui/Cart`, replace `AddToCartStub` | `/cart` view; PDP/list add; header badge |
| **App** | `src/app/(storefront)/cart` | Thin route → view |
| **Client bootstrap** | browser token / IP | Legacy `IpService` parity (`browsertoken`) |

**Flow:** UI → Presenter → Use case → Repository (interface) → Data REST → API  
UI must **not** call axios/fetch or domain use cases directly.

---

## Critical business rules (preserve)

1. Cart keyed by `{userId}-{ipAddress}`; guests use `userId = 0`
2. LS keys: `cart`, `browsertoken` (same strings as legacy)
3. Add-time mixing ≠ checkout mixing (both must match legacy)
4. Checkout: no layaway + pre-order mix; layaway/pre-order require login
5. `preOrderLayaway` lines cannot be duplicated on add
6. Line unit: pre-order DP when `preOrder || (preOrderLayaway && !isLayAway)`, else sale/price; optional rush fee
7. Admin cannot mutate cart / checkout
8. Stock validate via `POST /cart/validatecartbyuserid` before shipping

### API surface (unchanged)

| Method | Path |
|--------|------|
| POST | `/cart/createcart` |
| GET | `/cart/getcartbyuseridandip/{userId}-{ip}` |
| PATCH | `/cart/updatecart` |
| POST | `/cart/deletecartbyid` |
| DELETE | `/cart/deleteitemincartbyemail/{email}` |
| POST | `/cart/validatecartbyuserid` |
| PATCH | `/cart/updatecartuser` |
| POST | `/cart/validateproduct` |

Layaway config (product controller): `GET /product/getActiveLayAway/`, `GET /product/getlayawaydates/`

---

## Waves

### Wave 0 — Foundation
- This playbook + [`checklist.md`](./checklist.md)
- Client IP / `browsertoken` bootstrap (jsonip + synthetic fallback)
- Shared cart LS key constants
- `/cart` placeholder + header link ready for badge
- Confirm RestClient can hit `/cart/*` with existing auth headers
- Update `.cursor/docs/migration-progress.md` (Cart in-progress)

### Wave 1 — Domain
- Models + repository interfaces + use cases + pure helpers
- No React, no REST URLs in domain

### Wave 2 — Data + IoC
- REST against unchanged endpoints; DTO → domain mappers
- Layaway config clients
- `CartModule` registered in container

### Wave 3 — Presentation
- Cart page presenter (lines, totals, rush/layaway, checkout gate)
- Add-to-cart presenter API for PDP/cards
- Header cart-count presenter / shared store if needed

### Wave 4 — UI + App
- Real `/cart` UI (references when available; else legacy behavior + Catalog visual language)
- Replace `AddToCartStub`; header badge = sum of quantities
- Checkout → shipping route handoff (placeholder OK)
- Login merge when Auth session available

### Wave 5 — Verify
- Guest CRUD by IP; login merge; mixing + stock + gates; rush + installments
- Mark Cart done in `.cursor/docs/migration-progress.md`

**Do not** start Wave 4 until Waves 1–3 exist.  
**Do not** implement full Checkout / Payment inside Cart waves.

---

## Copy-paste orchestrator prompt

```text
Follow AGENTS.md, .cursor/docs/cart/orchestration.md, .cursor/docs/cart/checklist.md.
Use clean architecture in byzmo-v2/web (same patterns as Catalog).
Preserve Byzmo /api/cart contracts and legacy mixing/login rules.
Execute ONLY Wave N: <name>. Do not implement later waves.
Update checklist and .cursor/docs/migration-progress.md when done.
```

---

## Done when

Guest and logged-in cart CRUD works on live API, mixing/login/stock rules match legacy, Add to bag is real, header badge updates, checkout CTA hands off to shipping module, progress doc updated.
