# Migration workflow

How we migrate one storefront module on `byzmo-v2/web`.

Related: [`README.md`](./README.md) · [`migration-progress.md`](./migration-progress.md) · [`module-inventory.md`](./module-inventory.md) · [`AGENTS.md`](../../AGENTS.md)

---

## Big picture

```
Pick module → Wave 0 (docs + foundation) → Waves 1–4 (code by layer) → Wave 5 (verify) → mark done
```

| Doc | Purpose |
|-----|---------|
| **This file** | How the migration process works |
| `migration-progress.md` | What’s active / done / deferred |
| `module-inventory.md` | Legacy Angular map (routes, services) |
| `<module>/orchestration.md` | Playbook for **one** module (scope, APIs, waves) |
| `<module>/checklist.md` | Checkboxes for that module’s waves |
| `byzmo-frontend-layers.mdc` | Code layer rules only (not this workflow) |

---

## Starting a new module

1. Confirm in chat (or `migration-progress.md`) which module is next.
2. **Wave 0** — analyze legacy, then create:
   - `.cursor/docs/<module>/orchestration.md`
   - `.cursor/docs/<module>/checklist.md`
3. Add the module to [`README.md`](./README.md) modules table.
4. Set **Current focus** in [`migration-progress.md`](./migration-progress.md) to that module (`in-progress`).
5. Only after Wave 0 docs exist: implement later waves when asked (`wave 1`, `wave 2`, …).

Do **not** invent a second playbook format — copy the structure below (same as Catalog / Cart / Auth / Checkout / Account).

---

## Where orchestration + checklist content comes from

Agents (and humans) fill Wave 0 docs from:

1. **Legacy** `byzmo/web` — routes, components, services, validators  
2. **Inventory** — [`module-inventory.md`](./module-inventory.md)  
3. **Prior modules** — e.g. Auth deferred signup → Account; Checkout deferred gateways → Payments  
4. **Your chat constraints** — skip Payments, preserve APIs, etc.  
5. **`AGENTS.md`** — one module at a time; never edit legacy  

There is no separate “content schema” file beyond this workflow + the outlines below.

---

## Standard waves (every module)

| Wave | Name | Typical output |
|------|------|----------------|
| **0** | Foundation | `orchestration.md` + `checklist.md`; routes/placeholders; endpoint consts; progress → in-progress |
| **1** | Domain | `domain/<Module>/` models, helpers, repo interfaces, use cases (no React/URLs) |
| **2** | Data + IoC | REST DTOs/mappers/repos; `<Module>Module` in container |
| **3** | Presentation | Presenters + view models |
| **4** | UI + App | Real forms/pages; wire session/gates |
| **5** | Verify | Live/smoke script or checklist evidence; module → **done** in progress |

Say **`wave N`** (or “apply next todo”) to run **only** that wave.

After each wave:

- Check off items in `<module>/checklist.md`
- Append a changelog line in `migration-progress.md`
- Update Current focus notes

---

## `orchestration.md` outline

Create `.cursor/docs/<module>/orchestration.md` with roughly:

```markdown
# <Module> Execution + Frontend Orchestrator

How to run **storefront <Module>** on `byzmo-v2/web` …

Related: checklist · migration-progress · module-inventory · sibling modules

## Scope
- In scope (routes / features table)
- Deferred (other modules)

## Prerequisites
## Architecture map (layers + paths)
## Critical business rules (preserve)
## API surface (method + path + notes)
## Waves
### Wave 0 — Foundation
### Wave 1 — Domain
### Wave 2 — Data + IoC
### Wave 3 — Presentation
### Wave 4 — UI + App
### Wave 5 — Verify
## Copy-paste orchestrator prompt
## Done when
```

---

## `checklist.md` outline

Create `.cursor/docs/<module>/checklist.md` with roughly:

```markdown
# <Module> Migration Checklist

Related: orchestration · migration-progress · module-inventory

**Status key:** `[ ]` todo · `[x]` done

## 0. Foundation (Wave 0)
- [ ] orchestration + this checklist
- [ ] …

## 1. Domain (Wave 1)
- [ ] …

## 2. Data + IoC (Wave 2)
- [ ] …

## 3. Presentation (Wave 3)
- [ ] …

## 4. UI + App (Wave 4)
- [ ] …

## 5. Acceptance (Wave 5)
- [ ] …
- [ ] migration-progress → done

### Wave N evidence (optional)
- Script / live notes
```

---

## Running an existing module

1. Open `.cursor/docs/<module>/orchestration.md` + `checklist.md`
2. Execute **one wave** only
3. Update checklist + `migration-progress.md`
4. Stop and wait for the next wave request (unless the user asked for more)

UI references: [`ui/ui-brief.md`](./ui/ui-brief.md) · [`ui/references/`](./ui/references/).

Layer coding rules: [`.cursor/rules/byzmo-frontend-layers.mdc`](../rules/byzmo-frontend-layers.mdc).
