# Byzmo Migration Project

## Goal
Migrate the existing Angular e-commerce frontend to a modern Next.js/React application with an improved UI.

## Projects
- `byzmo/web`: Existing frontend (Angular 8). Reference only.
- `byzmo/api`: Existing API source, if available.
- `byzmo-v2/web`: New frontend development target (Next.js/React template).

## Rules
1. Preserve existing API endpoints and contracts.
2. Preserve business logic, validations, and user workflows.
3. Improve the UI without removing existing functionality.
4. Never modify the legacy application.
5. Work on one module at a time.
6. Analyze before implementing.
7. Document important findings and migration progress. Follow `.cursor/docs/workflow.md` (Wave 0 creates per-module `orchestration.md` + `checklist.md`; update `.cursor/docs/migration-progress.md`). Index: `.cursor/docs/README.md`.
8. Ask before making major architectural changes.