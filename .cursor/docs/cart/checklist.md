# Cart Migration Checklist

Related: [`orchestration.md`](./orchestration.md) · [`../migration-progress.md`](../migration-progress.md) · [`../module-inventory.md`](../module-inventory.md)

**Status key:** `[ ]` todo · `[x]` done

## 0. Foundation (Wave 0)
- [x] Cart orchestration + this checklist
- [x] Browser IP / `browsertoken` bootstrap (legacy `IpService` parity)
- [x] Cart LS key constants (`cart`, `browsertoken`, `MIN_INSTALLMENT`)
- [x] `/cart` placeholder + header cart link
- [x] RestClient `/cart/*` wired (Wave 2 repos; live smoke in Wave 5)

## 1. Domain (Wave 1)
- [x] Cart + layaway models
- [x] `ICartRepository` (+ layaway config port)
- [x] Use cases: load / add / update / remove / validate / merge-on-login
- [x] Helpers: line totals, mixing (add + checkout), layaway deposit/schedule, cart user-ip key

## 2. Data + IoC (Wave 2)
- [x] REST endpoints, DTOs, mappers, repos
- [x] Layaway config (`getActiveLayAway`, `getlayawaydates`)
- [x] `CartModule` + symbols in container

## 3. Presentation (Wave 3)
- [x] Cart page presenter + view models
- [x] Add-to-cart presenter API
- [x] Header cart-count presenter (+ merge-on-login)

## 4. UI + App (Wave 4)
- [x] `/cart` page UI
- [x] Replace `AddToCartStub` (PDP + cards)
- [x] Header badge = quantity sum
- [x] Checkout CTA → shipping handoff (`/shipping/information` placeholder)
- [x] Login merge presenter ready; Auth UI still deferred (guest `userId=0`)

## 5. Acceptance (Wave 5)
- [x] Guest add / update / delete keyed by IP
- [x] Login merges guest cart (`PATCH /cart/updatecartuser` — guest → user by IP)
- [x] Mixing + stock + layaway/pre-order login gates (domain helpers + API validate)
- [x] Rush fee + installment schedule numbers (helpers vs Azure active layaway 20%/5)
- [x] No full checkout/payment implementation in this slice
- [x] `.cursor/docs/migration-progress.md` Cart → done

### Wave 5 evidence (2026-10-30, Azure `:5549/api`)
- Layaway config: id=7, payment day 5, max installments 5, 20% deposit
- Guest CRUD: create → get `0-{ip}` → update qty → validate product/cart → merge to user 999999 → delete (product 21)
- Domain script: `npx tsx scripts/cart-wave5-verify.mts` (all assertions passed)
- Jest unit file added at `src/domain/Cart/helpers/cartHelpers.spec.ts` (jest runner currently broken in this env via wrap-ansi ESM)
