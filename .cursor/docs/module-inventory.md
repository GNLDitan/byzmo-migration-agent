# Legacy Module Inventory

Source: `byzmo/web` (Angular ~8.2). API contracts preserved via existing backend (`byzmo/api`).

See also: [`README.md`](./README.md) · [`migration-progress.md`](./migration-progress.md)

## Architecture

| Item | Notes |
|------|--------|
| App shape | Monolithic `AppModule`; only `ProfileDashboardModule` is lazy-loaded |
| State | `DataService` (BehaviorSubjects) |
| HTTP | `AppInterceptor` → `{environment.serverUrl}` + Bearer token |
| UI stack | Bootstrap 4, ng-bootstrap, Material 8, PrimeNG 7 |

## Storefront modules

| Module | Routes (key) | Main components | Services |
|--------|--------------|-----------------|----------|
| Catalog | `/home`, `/products`, `/products/:id`, `/features` | Home, Products, ShopProducts, ProductFilter, Landing, Category | Product, Category, Landing, Currency, Filter |
| Cart | `/cart` | Cart | Cart, LayAway, Product |
| Checkout | `/shipping/information` → `shipping-method` → `payment` | ShippingDetails, ShippingMethod, Payment | Shipping, Payment, Order, Cart |
| Payments | `/order/payment-gateway/...` (paypal, card, paymongo, gcash, grabpay, layaway, pre-order) | PaymentGateway + gateway children | Payment, Paymongo, Braintree, Order |
| Auth / Login | No `/login` route — modal in Navigation (also PDP, shipping, cart); guest: `/payment/auth/login-guest/:id` | Navigation (login modal), LoginGuest, Signup | User (`authenticate`), Auth, social login (Google/Facebook) |
| Account | `/signup`, `/forgot/*`, `/profile/*` | Signup, ForgotPassword*, ProfileDashboard, Information, Addresses | User, Auth, ResetToken |
| Orders (customer) | `/client-order`, `/client-order/client-order-info/:id`, reviews, complete | ClientOrder*, CompleteOrder/Payment, CustomerOrderReview | Order, ProductReview |
| Content | `/blog`, `/blog/content/:id/:title`, static pages | BlogPage*, AboutUs, Terms, Privacy, policies | Blog |
| Loyalty / extras | `/loyalty-discount`, `/wishlist`, subscribe/unsubscribe | LoyaltyDiscount, UserWishlist, Subscribe* | Loyalty, User |

## Admin modules (`/administration`)

| Module | Routes (key) | Services |
|--------|--------------|----------|
| Dashboard | `/administration/dashboard` | Dashboard |
| Products | `product`, `product/add`, `product/:id`, inventory, notification | Product, File |
| Categories | `category-maintenance/...` | Category, Product |
| Orders | `orders`, `orders/:id` | Order, Payment |
| Shipping / Payment methods / Discounts / Layaway / Loyalty | respective CRUD routes | Shipping, Payment, Product, Loyalty |
| Blog / Landing | `blog/*`, `admin-landing/*` | Blog, Landing, File |
| User admin | `user-admin/*` | User |
| Reports | order/sales/stock/tags/customers/subscriptions, etc. | Report |

## Shared / cross-cutting

- Guards (~20): Admin, product/blog/shipping/payment/order, layaway/pre-order/full-payment, GCash success
- Validators: `ValidatorService` (email, password, mobile, numeric, installment)
- Models: `src/app/classes/*` (~78)
- Utils: `app.utils.ts` (cart LS keys, payment types, report type IDs, currency base PHP)

## API surface (by service prefix)

| Prefix | Domain |
|--------|--------|
| `/product` | Catalog, inventory, tags, layaway config, discounts, categories |
| `/cart` | Cart CRUD + validation |
| `/shipping` | Methods + checkout shipping details |
| `/order` | Orders, schedules, ratings |
| `/payment` | Methods, fees, complete/layaway/preorder/GCash |
| `/paymongo`, `/braintree` | Online payments |
| `/user` | Auth, profile, addresses, wishlist, notifications, subscriptions |
| `/loyalty` | Vouchers / loyalty discounts |
| `/blog`, `/landing`, `/file`, `/currency` | Content, uploads, FX |
| `/dashboard`, `/report` | Admin analytics |
| `/email/*` | Transactional emails |

## Critical business rules (preserve)

- Sale price when `onSale`; optional rush fee
- Stock quantity enforcement; cart validation before checkout
- Layaway + pre-order cannot mix; both require logged-in user
- Layaway: non-refund deposit % + installment schedule (`LayAwayService`)
- Product flags: `preOrder`, `isLayAway`, `preOrderLayaway`, `hasRushFee`
