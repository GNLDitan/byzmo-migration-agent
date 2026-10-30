# Catalog Execution + Frontend Orchestrator

How to run Catalog migration on `byzmo-v2/web` **using the clean-architecture layers** (not ad-hoc pages).

Related: [`checklist.md`](./checklist.md) · [`../ui/ui-brief.md`](../ui/ui-brief.md) · [`../migration-progress.md`](../migration-progress.md) · boilerplate `byzmo-v2/web/README.md`

---

## Prerequisites (before coding)

1. Drop your UI basis in `.cursor/docs/ui/references/` and mark rows in `.cursor/docs/ui/ui-brief.md`
2. Byzmo API reachable; set `API_URL` (+ image base URLs)
3. Keep checklist open — orchestrator works **one wave at a time**

---

## Architecture map (Catalog)

Mirror existing features (`Clients` / `Packages`). New bounded context: **Catalog** (and thin **Landing** if needed).

| Layer | Path | Catalog work |
|-------|------|----------------|
| **Domain** | `src/domain/Catalog/` | Models (`Product`, `Category`, …), repository **interfaces**, use cases (fetch list, by linkName, related, categories, home features) |
| **Data** | `src/data/network/rest/v1/Catalog/` (+ Landing/Currency as needed) | REST clients hitting legacy paths (`/product/...`, `/landing`, `/currency/...`), DTOs, mappers → domain |
| **IoC** | `src/ioc/` | Bind Catalog repositories / use cases / presenters |
| **Presentation** | `src/presentation/Catalog/` | Presenters + view models for Home / Products / PDP |
| **UI** | `src/ui/Home`, `Products`, `ProductDetail`, shell | Views/styles from **your** references; call presenters only |
| **App** | `src/app/...` | Thin routes: `/`, `/products`, `/products/[linkName]` (and `/home` redirect if needed) |
| **DataStore** | `src/dataStore/` | Only if list/filter state needs shared client store |

**Flow (required):** UI → Presenter → Use case → Repository (interface) → Data REST → API  
UI must **not** call axios/fetch or domain use cases directly.

**Layer rules** (from template): Domain has no outward deps; Data implements domain repos; Presentation uses Domain; UI uses Presentation (+ DataStore).

---

## Orchestrator = wave prompts

Treat the agent as orchestrator: **one wave per chat/task**, then check off the checklist and update `.cursor/docs/migration-progress.md`.

### Wave 0 — Foundation
- Env `API_URL` + image paths  
- Shared HTTP client / auth header hook (Bearer token later)  
- Storefront shell layout (nav/footer) empty links OK  

### Wave 1 — Domain (Catalog)
- Models + repository interfaces + use cases only  
- No React, no REST URLs in domain  

### Wave 2 — Data
- Implement REST against **unchanged** Byzmo endpoints (see checklist §1)  
- Map API JSON → domain models  
- Register IoC bindings  

### Wave 3 — Presentation
- Presenters for Home, Product list, PDP  
- Sale-price rule, linkName routing prep  

### Wave 4 — UI + App routes
- Build screens from `.cursor/docs/ui/references/`  
- Wire `app/` pages to views  
- Stub Add to cart  

### Wave 5 — Verify
- Run against live/local API  
- Checklist acceptance  
- Mark Catalog in `.cursor/docs/migration-progress.md`  

**Do not** start Wave 4 until Waves 1–3 exist (keeps architecture intact).  
**Do not** mix Cart/Auth into Catalog waves.

---

## Copy-paste orchestrator prompt (per wave)

```text
Follow AGENTS.md, .cursor/docs/catalog/checklist.md, .cursor/docs/catalog/orchestration.md.
Use clean architecture in byzmo-v2/web (same patterns as domain/Clients + data/.../Clients + presentation/Clients + ui/...).
UI basis: .cursor/docs/ui/references/ (if empty, structure only — no invented visual system).
Execute ONLY Wave N: <name>. Do not implement later waves.
Update checklist checkboxes and .cursor/docs/migration-progress.md when done.
```

Replace `Wave N` with `0`–`5` as you go.

---

## Suggested Cursor setup

| Piece | Role |
|-------|------|
| `.cursor/docs/catalog/orchestration.md` | This playbook (order + layers) |
| `.cursor/docs/catalog/checklist.md` | Done/not-done truth |
| `.cursor/docs/ui/references/` | Visual truth |
| `.cursor/rules/byzmo-storefront-ui.mdc` | UI follows your mocks |
| Optional: always-apply or `byzmo-v2/web/**` rule | “Respect layer import restrictions / clean arch” |

You don’t need a separate workflow engine. **Checklist + waves + one-wave prompts** is the orchestrator.

---

## Done when

Home, `/products`, and PDP work on real API, UI matches your references, no cart/order/payment calls, progress doc updated. Then next module gets its own folder under `.cursor/docs/` (checklist + waves).
