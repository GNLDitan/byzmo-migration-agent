# Catalog Migration Checklist

Related: [`orchestration.md`](./orchestration.md) · [`../migration-progress.md`](../migration-progress.md) · [`../ui/ui-brief.md`](../ui/ui-brief.md)

**Status key:** `[ ]` todo · `[x]` done

## 0. Foundation
- [x] Env API + image paths
- [x] HTTP client + Bearer token/guesttoken
- [x] Domain types
- [x] Storefront shell

## 1. APIs (Wave 2)
- [x] Landing, home features, categories, products list/by category/by id/by linkName, related, currency

## 2. Pages / UI (Wave 4 — from `.cursor/docs/ui/references/`)
- [x] Home (`home.jpg`)
- [x] Products (`products.jpg`)
- [x] PDP (`product-details.jpg`)
- [x] Sale price + image paths + linkName routing
- [x] Add to cart stubbed
- [x] Manual mobile/desktop check (Wave 5)

## 4. Acceptance (Wave 5)
- [x] Home loads live landing + featured data
- [x] `/products` lists with filter/category
- [x] PDP loads product + related
- [x] Prices render in ₱
- [x] No cart/order/payment API calls in this slice
