# Byzmo migration docs

Top-level trackers stay here. Each storefront module keeps its **orchestrator** + **checklist** in its own folder.

**Start here for process:** [`workflow.md`](./workflow.md)

## Layout

```
.cursor/docs/
  README.md                 ← this index
  workflow.md               ← how migration waves / docs work
  migration-progress.md     ← overall status / changelog
  module-inventory.md       ← legacy Angular module map
  catalog/
    orchestration.md
    checklist.md
  cart/
    orchestration.md
    checklist.md
  auth/
    orchestration.md
    checklist.md
  checkout/
    orchestration.md
    checklist.md
  account/
    orchestration.md
    checklist.md
  orders/
    orchestration.md
    checklist.md
  ui/
    ui-brief.md
    references/             ← user-provided mockups
```

## Modules

| Module | Orchestrator | Checklist | Status |
|--------|--------------|-----------|--------|
| Catalog | [orchestration.md](./catalog/orchestration.md) | [checklist.md](./catalog/checklist.md) | done |
| Cart | [orchestration.md](./cart/orchestration.md) | [checklist.md](./cart/checklist.md) | done |
| Auth | [orchestration.md](./auth/orchestration.md) | [checklist.md](./auth/checklist.md) | done |
| Checkout | [orchestration.md](./checkout/orchestration.md) | [checklist.md](./checkout/checklist.md) | done |
| Account | [orchestration.md](./account/orchestration.md) | [checklist.md](./account/checklist.md) | done |
| Customer orders | [orchestration.md](./orders/orchestration.md) | [checklist.md](./orders/checklist.md) | in-progress |
| Payments | — | — | deferred |

## How to run a module

See [`workflow.md`](./workflow.md). Short version:

1. Open that module’s `orchestration.md` + `checklist.md`
2. Work **one wave at a time**
3. Update the checklist and [`migration-progress.md`](./migration-progress.md) when a wave finishes

UI work: [`ui/ui-brief.md`](./ui/ui-brief.md) + [`ui/references/`](./ui/references/).
