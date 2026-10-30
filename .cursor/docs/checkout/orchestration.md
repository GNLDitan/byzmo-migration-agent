# Checkout Execution + Frontend Orchestrator

How to run **storefront Checkout** (shipping → method → payment / place order) on `byzmo-v2/web` using clean-architecture layers (same pattern as Catalog / Cart / Auth).

Related: [`checklist.md`](./checklist.md) · [`../migration-progress.md`](../migration-progress.md) · [`../module-inventory.md`](../module-inventory.md) · [`../cart/orchestration.md`](../cart/orchestration.md) · [`../auth/orchestration.md`](../auth/orchestration.md)

---

## Scope

Legacy nested routes under shipping:

| Step | URL | Legacy component |
|------|-----|------------------|
| 1 — Details | `/shipping/information` | ShippingDetails |
| 2 — Method | `/shipping/information/shipping-method` | ShippingMethod |
| 3 — Payment | `/shipping/information/shipping-method/payment` | Payment |

This module covers:

- Checkout shell (cart summary sidebar, discount, totals)
- Shipping address form (+ logged-in address book)
- Shipping method + insurance
- Payment **method selection**
- `createshippingdetails` → `createorder` → order emails
- Redirect **decision** to gateway URL or offline complete page

**Deferred to Payments module**

- `/order/payment-gateway/*` UIs (PayPal, Braintree, Paymongo, GCash, GrabPay)
- `completepayment`, webhooks, success/failure pages
- Guest payment login (`/payment/auth/login-guest/:id`)
- Layaway / pre-order **payment** routes

**Deferred to Account**

- Full address CRUD UX beyond checkout address book

Cart / Auth stay sources of truth for cart lines and `userId` / login modal.

---

## Prerequisites

1. Catalog + Cart + Auth done
2. Byzmo API reachable (`API_URL`)
3. Preserve shipping / order / payment-list contracts
4. One wave at a time

---

## Architecture map (Checkout)

| Layer | Path | Checkout work |
|-------|------|---------------|
| **Domain** | `src/domain/Checkout/` | Draft models; shipping/payment/order shapes; pure totals helpers |
| **Data** | `src/data/network/rest/v1/Checkout/` | REST: shipping, payment methods, order create, discount, emails |
| **IoC** | `src/ioc/Checkout/` | Bind repos / use cases / presenters |
| **Presentation** | `src/presentation/Checkout/` | Shell + step presenters / VMs |
| **UI** | `src/ui/Checkout` | Wizard views + shell |
| **Client session** | `ui/common/checkout` | `CheckoutDraft` context (replaces legacy `DataService` checkout streams) |

**Flow:** UI → Presenter → Use case → Repository → REST → API

---

## Critical business rules (preserve)

1. Cart entry: reuse Cart `prepareCheckout` (mixing + login gates + stock validate)
2. Domestic vs intl shipping: `countryCode === 'PH'` vs international (`LOCAL_COUNTRY_CODE`)
3. Step 1 requires `facebookName` (legacy form)
4. Totals: shipping breakdown + insurance + discount + payment transaction fee (client math)
5. GCash / GrabPay: block when amount-to-pay / installment / DP ≤ ₱100 (`hasMinimumAmount`)
6. Order create on payment submit **before** gateway: validateproduct → createshippingdetails → createorder → emails
7. `createorder` clears cart server-side — refresh header badge after success
8. Online methods → stub redirect to Payments routes; offline → complete-order placeholder

### API surface (this module)

| Method | Path | Notes |
|--------|------|-------|
| POST | `/shipping/getshippinglistrange` | Method list |
| POST | `/shipping/createshippingdetails` | Before order |
| PATCH | `/shipping/updateshippingdetails` | Optional |
| POST | `/payment/getcheckoutpaymentmethodlistrange` | Step 3 list |
| POST | `/order/createorder` | Place order |
| POST | `/product/getalldiscountbycode` | Sidebar discount |
| POST | `/email/sendorderemail` | Fire-and-forget |
| POST | `/email/sendadminorder` | Fire-and-forget |
| POST | `/cart/validateproduct` | Already Cart | Pre-submit stock |

User addresses: `GET /user/getuseraddressbyuserid/{id}` (Auth/User).

---

## Waves

### Wave 0 — Foundation
- This playbook + checklist
- Storefront routes for all 3 steps + checkout shell placeholders
- `CheckoutSession` / draft context (client)
- `CheckoutEndpoints` consts
- Domain consts (`LOCAL_COUNTRY_CODE`, GCash min)
- Update `migration-progress.md` (Checkout in-progress)

### Wave 1 — Domain
- Models + repo interfaces + pure helpers (totals, insurance, fees, step validation)
- Use cases (load methods, save details, place order, discount) — no React/URLs in domain

### Wave 2 — Data + IoC
- REST clients + mappers + `CheckoutModule`

### Wave 3 — Presentation
- Shell / details / method / payment presenters + VMs

### Wave 4 — UI + App
- Real step UIs + shell totals/discount; submit pipeline; login modal reuse

### Wave 5 — Verify
- Live shipping list + payment methods + place-order happy path (offline or stub gateway)
- Mark Checkout done; Payments remains separate

**Do not** implement gateway UIs inside Checkout waves.

---

## Copy-paste orchestrator prompt

```text
Follow AGENTS.md, .cursor/docs/checkout/orchestration.md, .cursor/docs/checkout/checklist.md.
Use clean architecture in byzmo-v2/web (same patterns as Catalog/Cart/Auth).
Preserve shipping/order/payment-list contracts. Gateways stay in Payments module.
Execute ONLY Wave N: <name>. Do not implement later waves.
Update checklist and .cursor/docs/migration-progress.md when done.
```

---

## Done when

Users can complete shipping → method → payment method, place order via API, see cart cleared, and be handed off to Payments or offline complete placeholder; progress doc updated.
