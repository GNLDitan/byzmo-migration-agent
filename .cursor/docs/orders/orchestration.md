# Customer Orders Execution + Frontend Orchestrator

How to run **storefront Customer Orders** (My Orders list, detail, cancel, reviews) on `byzmo-v2/web` using clean-architecture layers (same pattern as Catalog / Cart / Auth / Checkout / Account).

Related: [`checklist.md`](./checklist.md) · [`../migration-progress.md`](../migration-progress.md) · [`../module-inventory.md`](../module-inventory.md) · [`../account/orchestration.md`](../account/orchestration.md) · [`../checkout/orchestration.md`](../checkout/orchestration.md)

---

## Scope

| Surface | URL | Legacy component |
|---------|-----|------------------|
| My Orders | `/client-order` | ClientOrder |
| Order detail | `/client-order/client-order-info/:id` | ClientOrderInfo (`:id` = **securityId**) |
| Rate & review | `/client-order/reviews/:orderid` | CustomerOrderReview (`:orderid` = **securityId**) |

This module covers:

- Paginated order list for the logged-in customer (`getordersbyuserid` with `{userId}-{ip}`)
- Order detail by `securityId` (line items, cost breakdown, layaway/pre-order schedule **display**)
- Cancel order when fulfillment is Unfulfilled (`statusId === 1`) + cancel email + `updateorder`
- Product ratings after Paid (`paymentStatusId === 3`) via `createorderproductrate`
- Pay Now **visibility / routing stubs** → existing Payments / complete-order placeholders (no gateway UIs)

**Reuse**

- Auth session (token / email / userId) + Cart IP / browsertoken patterns for list filter key
- Checkout `ORDER_COMPLETE` / `PAYMENT_GATEWAY` route helpers for Pay Now handoff
- Account profile nav can deep-link to My Orders

**Deferred**

| Item | Module |
|------|--------|
| `/order/payment-gateway/*` UIs | Payments (already deferred) |
| `/order/compete/:id`, `/order/payment-complete/:id` | Checkout stubs / Payments |
| `POST /order/completeorder` | Dead in legacy UI — skip unless API requires |
| `/client-order-info-view/:id` email-style page | Optional later (email deep links) |
| PDP product reviews (`ProductReviewService`) | Catalog extras |
| Admin order list / reports | Admin |

**Payments** remains deferred; Pay Now buttons may navigate to existing gateway stubs only.

---

## Prerequisites

1. Catalog + Cart + Auth + Checkout + Account done
2. Byzmo API reachable (`API_URL`)
3. Preserve `/order/*` + cancel email contracts
4. One wave at a time

---

## Architecture map (Orders)

| Layer | Path | Orders work |
|-------|------|-------------|
| **Domain** | `src/domain/Orders/` | Order / review models; status helpers; pay-now / cancel rules; repo interfaces; use cases |
| **Data** | `src/data/network/rest/v1/Orders/` | REST DTOs, mappers, repos |
| **IoC** | `src/ioc/Orders/` | Bind repos / use cases / presenters |
| **Presentation** | `src/presentation/Orders/` | List / detail / review presenters + VMs |
| **UI** | `src/ui/Orders` | My Orders, detail, review views |
| **Session** | Auth + Cart IP | Gate list data; build `{userId}-{ip}` search key |

**Flow:** UI → Presenter → Use case → Repository → REST → API

---

## Critical business rules (preserve)

1. No Angular route guard; list stays empty without token (v2 may soft-prompt login — prefer open login modal, match empty-list behavior)
2. List filter: `searchInput = "{userId}-{ipAddress}"`, high `limit` (legacy `99999999`)
3. Detail / review URLs use **`securityId`**, not numeric `id`
4. Fulfillment: 1 Unfulfilled, 2 Processing, 3 Shipped, 4 Fulfilled, 5 Cancelled
5. Payment: 1 Unpaid, 2 Partial, 3 Paid
6. Cancel only when `statusId === 1`; reason from preset `REASON_FOR_CANCEL`; set `statusId = 5` + `orderReason`; email then PATCH
7. Review entry when `paymentStatusId === 3`; one rating per cart line (`hasReview`); stars required
8. Pay Now enable (list): unpaid + not cancelled + status ≤ 3; disable Bank/OTC (except Online Banking); `paymongoStatus === 2` → Processing
9. Tracking link when `statusId === 3` and tracking URL present
10. Layaway / pre-order: display schedules + pending totals; installment Pay Now → Payments stubs only

### API surface (this module)

| Method | Path | Notes |
|--------|------|-------|
| POST | `/order/getordersbyuserid` | Body `FilterSetting` (`limit`, `searchInput`) |
| GET | `/order/getorderbysecurityid/{securityId}` | Detail + review load |
| PATCH | `/order/updateorder` | Cancel (and any status patch) |
| POST | `/order/createorderproductrate` | Line rating |
| POST | `/email/sendorderemail` | Cancel notification (legacy `adminSendEmail`) |

Hydrate user: Auth `getuserbyemail` (already Account/Auth).  
Do **not** use `ProductReviewService` / `/productreview/*` for this slice.

---

## Waves

### Wave 0 — Foundation
- This playbook + checklist
- Storefront routes + placeholder pages for list / detail / reviews
- `OrdersEndpoints` consts
- Header: My Orders when logged in
- Update `migration-progress.md` (Orders in-progress; Payments deferred)

### Wave 1 — Domain
- Models + status/cancel/pay-now helpers + repo interfaces + use cases (no React / URLs)

### Wave 2 — Data + IoC
- REST clients + mappers + `OrdersModule`

### Wave 3 — Presentation
- List / detail / review presenters + VMs

### Wave 4 — UI + App
- Real My Orders table, detail (cancel + schedules), review form; Pay Now → stubs

### Wave 5 — Verify
- Live list/detail/cancel/review smoke; mark Orders done; Payments still deferred

---

## Copy-paste orchestrator prompt

```text
Follow AGENTS.md, .cursor/docs/orders/orchestration.md, .cursor/docs/orders/checklist.md.
Use clean architecture in byzmo-v2/web (same patterns as Catalog/Cart/Auth/Checkout/Account).
New bounded context domain/Orders — reuse Auth session + Cart IP; do not build payment gateways.
Preserve /order getordersbyuserid, getorderbysecurityid, updateorder, createorderproductrate, sendorderemail.
Execute ONLY Wave N: <name>. Do not implement later waves.
Update checklist and .cursor/docs/migration-progress.md when done.
```

---

## Done when

Logged-in customers can list orders, open detail by securityId, cancel Unfulfilled orders, and submit product ratings on Paid orders; Pay Now hands off to existing stubs; progress doc updated; Payments still out of scope.
