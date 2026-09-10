# Import automotive template

Canonical standalone master: **`darkapoparka/cars-template-import`**. This repository is a reusable dealership design, not a dealer-specific project and not evidence that any sample business data is current.

**AI/agent entry point:** read `AGENTS.md`, then `TEMPLATE.md`, `docs/LEAD-BUILD.md`, and `docs/QA.md` before editing.

## Portfolio role
- Template key: `import`
- Role: **optional**
- Design position: import/sourcing specialist for dealers that actively sell the import journey
- Standard dealer offer: `auto-best + carwow + modern`
- `import` is an optional fourth design only for dealers with a real import/sourcing proposition.

## Rule of ownership
Improve this repository only when the task is a **shared template improvement**. For a **lead build**, make a clean copy and personalize the copy; never turn `main` into one dealer's site.

## Quick start
`npm ci`

Preview command: `npm run dev -- --host 127.0.0.1 --port 6464 --strictPort`

Entry route: `/`

See `TEMPLATE.md` for template-specific boundaries and `docs/LEAD-BUILD.md` for the complete lead workflow. Historical pre-split root docs are preserved under `docs/legacy/from-cars-2026-09-10/` for provenance only; they do not override the current instructions.
