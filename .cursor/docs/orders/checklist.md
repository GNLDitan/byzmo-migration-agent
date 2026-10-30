# Customer Orders Migration Checklist

Related: [`orchestration.md`](./orchestration.md) · [`../migration-progress.md`](../migration-progress.md) · [`../module-inventory.md`](../module-inventory.md)

**Status key:** `[ ]` todo · `[x]` done

## 0. Foundation (Wave 0)
- [x] Orders orchestration + this checklist
- [x] Storefront routes: `/client-order`, detail, reviews
- [x] Placeholder pages
- [x] `OrdersEndpoints` consts
- [x] Header My Orders when logged in
- [x] Progress: Orders in-progress (Payments deferred)

## 1. Domain (Wave 1)
- [x] Models (order summary/detail, product rate, cancel reason)
- [x] Pure helpers (status labels, pay-now enable, cancel gate)
- [x] Repository interfaces + use cases
- [x] No React / URLs in domain

### Wave 1 evidence
- Script: `npx tsx scripts/orders-wave1-verify.mts` — status/pay-now/cancel/review gates + fetch/cancel/review use cases

## 2. Data + IoC (Wave 2)
- [ ] REST DTOs, mappers, repositories
- [ ] `OrdersModule` + symbols in container

## 3. Presentation (Wave 3)
- [ ] Orders list presenter + VM
- [ ] Order detail presenter (cancel) + VM
- [ ] Order review presenter + VM

## 4. UI + App (Wave 4)
- [ ] My Orders list UI
- [ ] Order detail + cancel UI + schedule display
- [ ] Review UI
- [ ] Pay Now → gateway / complete stubs only

## 5. Acceptance (Wave 5)
- [ ] Live list by userId-ip
- [ ] Detail by securityId
- [ ] Cancel Unfulfilled path
- [ ] Create product rate on Paid order
- [ ] No gateway / PDP reviews / admin in this slice
- [ ] `migration-progress.md` Orders → done
