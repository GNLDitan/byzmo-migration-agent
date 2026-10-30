# Checkout Migration Checklist

Related: [`orchestration.md`](./orchestration.md) · [`../migration-progress.md`](../migration-progress.md) · [`../module-inventory.md`](../module-inventory.md)

**Status key:** `[ ]` todo · `[x]` done

## 0. Foundation (Wave 0)
- [x] Checkout orchestration + this checklist
- [x] Storefront routes: details / method / payment
- [x] `CheckoutSession` draft context
- [x] Checkout shell + step placeholders
- [x] `CheckoutEndpoints` + domain consts
- [x] Progress: Checkout in-progress

## 1. Domain (Wave 1)
- [x] Models (ShippingDetails, ShippingMethod, payment method, Order draft)
- [x] Pure helpers (shipping/total breakdown, insurance, transaction fee, step gates)
- [x] Repository interfaces + use cases
- [x] No React / URLs in domain

### Wave 1 evidence
- Script: `npx tsx scripts/checkout-wave1-verify.mts` — shipping tiers, insurance, fees, totals, step gates, handoff

## 2. Data + IoC (Wave 2)
- [x] REST DTOs, mappers, repositories
- [x] `CheckoutModule` + symbols in container

### Wave 2 evidence
- Repos: shipping, checkout payment, order, discount, email, user address
- Live smoke: `getshippinglistrange` OK (10 methods); `getcheckoutpaymentmethodlistrange` 404 on current Azure → repo falls back to `getpaymentmethodlistrange` + enable filter

## 3. Presentation (Wave 3)
- [x] Checkout shell presenter (sidebar totals + discount)
- [x] Shipping details / method / payment presenters + VMs

## 4. UI + App (Wave 4)
- [x] Step 1 address form (+ address book / login modal)
- [x] Step 2 method + insurance
- [x] Step 3 payment methods + place order
- [x] Post-order cart badge refresh + redirect stubs

## 5. Acceptance (Wave 5)
- [x] Live shipping list + payment methods
- [x] Place order (createshippingdetails + createorder)
- [x] GCash/Grab min rule; guest vs logged-in path
- [x] No gateway implementation in this slice
- [x] `migration-progress.md` Checkout → done

### Wave 5 evidence (2026-10-30, Azure `:5549/api`)
- Script: `npx tsx scripts/checkout-wave5-verify.mts` — all assertions passed
- Shipping: `getshippinglistrange` → 10 methods; domestic id=2
- Payment: `getcheckoutpaymentmethodlistrange` 404 → fallback `getpaymentmethodlistrange` (6 enabled); offline Bank Deposit id=14
- Guest place-order → orderId 638, cart cleared, handoff `/order/compete/638`
- Logged-in place-order → orderId 639, cart cleared, handoff offline; address book GET OK
- Gateways: stub routes only (Payments module)
