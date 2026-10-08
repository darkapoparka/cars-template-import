# Import automotive template

This repository contains the reusable Import dealership design, not a dealer-specific project or evidence that sample business data is current. Preserve the existing desktop/mobile surfaces when making shared code improvements.

**Contributor entry point:** read `TEMPLATE.md`, `docs/ARCHITECTURE.md`, `docs/LEAD-BUILD.md`, and `docs/QA.md`. This published checkout does not contain a root `AGENTS.md`.

**Publication lineage:** `.template/source.json` records the inherited Cars source commit and `templates/import` export. Check that metadata before synchronizing changes: older ownership notes predate this publication. Review-branch changes must be reconciled with the publisher's editable source before republishing; neither a branch push nor a passing build approves a dealer release.

The owner-requested `pro` audit, checks, measurements, and local Codex handoff are in [PRO audit](docs/PRO-AUDIT.md). `main` and the separate Cars checkout are not modified by that review.

## Portfolio role

- Template key: `import`
- Role: Design 2 option in an Import trio
- Design position: import/sourcing specialist for dealers that actively sell the import journey
- Standard dealer offer: `auto-best + carwow + modern`
- Import is Design 2 in the intentional Import trio; it does not add a fourth design.

## Rule of ownership

Improve this repository only when the task is a **shared template improvement**. For a **lead build**, use canonical Cars clients/<slug>/ through its approved-release clone workflow; never personalize this master.

## Quick start

`npm ci`

Preview command: `npm run dev -- --host 127.0.0.1 --port 6790 --strictPort`

Entry route: `/`

See `TEMPLATE.md` for template-specific boundaries and `docs/LEAD-BUILD.md` for the complete lead workflow. Historical pre-split root docs are preserved under `docs/legacy/from-cars-2026-09-10/` for provenance only; they do not override the current instructions.

Current cross-repository ownership, approved releases, dealer-copy workflow and standalone/mounted limits: [Cars integration](docs/CARS-INTEGRATION.md).

## Implemented architecture and checks

See [Architecture](docs/ARCHITECTURE.md) for native public routing, token ownership, dealer configuration and the deliberately retained legacy/demo boundary. Run `npm run verify`, `npm run build` and `npm run test:e2e` for source qualification. Current implementation evidence and remaining release gates are in [Localization handoff](docs/localization/HANDOFF.md).
