# AGENTS.md — rendezvous-kit

Instructions for AI coding agents working in this repository.

## What This Is

TypeScript library for finding fair meeting points for N people. Pipeline: isochrone computation, polygon intersection, venue search (Overpass API), route matrix, fairness scoring. One runtime dependency (`geohash-kit`). Zero third-party dependencies.

## Repository Layout

```
src/
  types.ts              — All shared interfaces and types
  geo.ts                — Polygon geometry (intersection, area, bbox, centroid, circle)
  hull.ts               — Convex hull fast-path strategy
  engines/              — Routing engine adapters (Valhalla, ORS, GraphHopper, OSRM)
  venues.ts             — Overpass API venue search
  rendezvous.ts         — Main pipeline function
  validate.ts           — Input validation helpers
  index.ts              — Barrel re-export
examples/               — Runnable TypeScript examples
docs/                   — Guides + interactive demo page (GitHub Pages)
```

## Commands

```bash
npm install            # install dependencies
npm run build          # compile TypeScript → dist/
npm test               # run all tests (vitest)
npm run typecheck      # type-check without emitting
npm run bench          # performance benchmarks
```

## Before Committing

1. Run `npm test && npm run typecheck` — both must pass
2. Use conventional commit messages: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`
3. Do not add third-party runtime dependencies
4. Use British English in all text (favour, colour, licence, initialise, metre)
5. Add JSDoc to any new public exports
6. GeoJSON coordinates are `[longitude, latitude]` — never `[lat, lon]`

## Key Interfaces

- **`RoutingEngine`** — implement `computeIsochrone`, `computeRouteMatrix`, `computeRoute` to add a new engine
- **`RendezvousOptions`** — input to `findRendezvous` (participants, mode, time, venues, fairness)
- **`RendezvousSuggestion`** — output: venue, travel times per participant, fairness score

## Testing Strategy

Tests are co-located with source files (`*.test.ts`). Engine tests mock HTTP responses via `globalThis.fetch`. The `rendezvous.test.ts` tests the full pipeline with mock engines.

## Release

Automated via semantic-release on push to `main`. Work on feature branches; merge to main only when complete.
