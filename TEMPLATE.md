# Template reference — Import

## Identity
- Repository: `darkapoparka/cars-template-import`
- Key: `import`
- Portfolio role: **optional**
- Design position: import/sourcing specialist for dealers that actively sell the import journey
- Stack: SvelteKit + Vite (Auxero-derived)
- Primary entry: `/`
- Suggested standalone review port: `6464`

This is a **template master**, not a sendable dealer demo. The baseline intentionally preserves source/sample material for design fidelity; every lead copy requires a complete identity and content sweep.

## Install and run
```text
npm ci
npm run dev -- --host 127.0.0.1 --port 6464 --strictPort
```

## Primary personalization surface
- `src/lib/data/daynight.ts`
- `src/lib/data/daynight-listings.json`
- `src/lib/data/vehicles.ts`
- `src/lib/styles/`
- `static/assets/daynight/`
- `static/`

Do not assume these are the only identity consumers. Search every retained route, data module, metadata definition and static asset before declaring a skin complete.

## Representative QA routes
- `/`
- `/inventory`
- `/contact`
- `/sell-your-car`
- `/financing`
- `/import`

## Required checks
- `npm run check`
- `npm run build`

## Current constraints
Use as a fourth variant only when import, sourcing, transport or order-from-Europe is a real part of the dealer's offer. It overlaps with the showroom/Auxero family, so do not add it mechanically.

For `modern`, local preview also requires the environment documented in `docs/QA.md`; provider services remain unconfigured unless a lead task explicitly wires them. For `carwow`, use direct Vite for a selectable port because the inherited source dev wrapper fixes port 6517.

## Source lineage
Split on 2026-09-10 from the live working tree at `J:/cars/templates/import`. The split deliberately captured local working-tree changes, including changes newer than the `cars` repository HEAD. Historical root instructions were archived under `docs/legacy/from-cars-2026-09-10/`; use them only for provenance, never as current operating instructions.

## Portfolio policy
Standard showroom lead = three variants: `auto-best`, `carwow`, `modern`. Add `import` only when the dealer's real offer includes sourcing/import/transport/order-from-Europe or equivalent. Do not add a fourth design merely to increase the count.
