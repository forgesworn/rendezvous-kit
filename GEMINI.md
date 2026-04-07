# GEMINI.md -- rendezvous-kit

TypeScript library for finding fair meeting points for N people using isochrone intersection, venue search, and fairness scoring.

## Commands

- `npm run build` -- compile TypeScript to `dist/`
- `npm test` -- run all tests (vitest, 161 tests across 9 files)
- `npm run typecheck` -- type-check without emitting
- `npm run bench` -- performance benchmarks
- `npm install` -- install dependencies

## Dependencies

Runtime dependencies only (zero third-party):

| Package | Description |
|---------|-------------|
| `geohash-kit` | Ours -- provides `pointInPolygon`, GeoJSON types, distance utilities |

## Structure

```
src/
  types.ts              -- All shared interfaces and types
  geo.ts                -- Polygon geometry (intersection, area, bbox, centroid, circle)
  hull.ts               -- Convex hull fast-path strategy
  engines/
    valhalla.ts         -- Valhalla adapter (isochrone + matrix)
    openrouteservice.ts -- ORS adapter (isochrone + matrix)
    graphhopper.ts      -- GraphHopper adapter (isochrone + matrix)
    osrm.ts             -- OSRM adapter (matrix only, no isochrone)
  venues.ts             -- Overpass API venue search
  rendezvous.ts         -- Main pipeline function (findRendezvous)
  validate.ts           -- Input validation helpers
  index.ts              -- Barrel re-export
examples/               -- Runnable TypeScript examples
docs/                   -- Guides and interactive demo page (GitHub Pages)
dist/                   -- Compiled output (generated)
```

## Subpath Exports

| Import path | Module |
|-------------|--------|
| `rendezvous-kit` | Main barrel -- all public exports |
| `rendezvous-kit/geo` | Polygon geometry utilities |
| `rendezvous-kit/venues` | Overpass venue search |
| `rendezvous-kit/rendezvous` | `findRendezvous` pipeline function |
| `rendezvous-kit/engines/valhalla` | `ValhallaEngine` |
| `rendezvous-kit/engines/openrouteservice` | `OpenRouteServiceEngine` |
| `rendezvous-kit/engines/graphhopper` | `GraphHopperEngine` |
| `rendezvous-kit/engines/osrm` | `OsrmEngine` |

## Engine Configuration

All engines are configured programmatically -- no env vars. Pass URLs and keys to constructors:

| Engine | Required | Notes |
|--------|----------|-------|
| `ValhallaEngine` | `baseUrl` | Self-hosted or `https://routing.trotters.cc` (L402-gated) |
| `OpenRouteServiceEngine` | `apiKey` | Free key from openrouteservice.org/dev; `baseUrl` optional |
| `GraphHopperEngine` | `baseUrl` | `apiKey` optional (needed for hosted GraphHopper) |
| `OsrmEngine` | `baseUrl` | Matrix only -- no isochrone support |

Overpass venue search uses public endpoints by default. Pass `overpassUrl` to `searchVenues()` to override.

## Key Interfaces

- `RoutingEngine` -- implement `computeIsochrone`, `computeRouteMatrix`, `computeRoute` to add a new engine
- `RendezvousOptions` -- input to `findRendezvous` (participants, mode, time, venues, fairness)
- `RendezvousSuggestion` -- output: venue, travel times per participant, fairness score

## Conventions

- **British English** -- favour, colour, behaviour, licence, initialise, metre
- **GeoJSON coordinate order: `[longitude, latitude]`** -- never `[lat, lon]`. This is the GeoJSON standard and is the most common source of bugs.
- **ESM-only** -- `.js` extensions in all imports, even for `.ts` source files
- **TypeScript strict mode** -- no `any`, no implicit returns
- **No third-party runtime dependencies** -- only `geohash-kit` (ours)
- **JSDoc** on all public exports

## Gotchas

- Coordinate order is `[lon, lat]` throughout -- matches GeoJSON spec but is the opposite of what many mapping APIs expect. Check carefully at engine adapter boundaries.
- `OsrmEngine` does not support isochrones. Do not pass it to pipelines that require isochrone computation.
- Engine tests mock `globalThis.fetch` -- no real HTTP calls are made in tests.
- The `rendezvous.test.ts` tests the full pipeline using mock engines, not real routing services.
- `hull.ts` is a convex hull fast-path strategy -- it is an optimisation, not a replacement for full isochrone intersection.

## Testing

9 test files co-located with source (`*.test.ts`), 161 tests total.

```bash
npm test          # vitest run (161 tests)
npm run test:watch  # vitest watch mode
```

## Release

Automated via semantic-release on push to `main`. Work on feature branches -- merge to main only when a logical chunk is complete to avoid version spam.

| Commit type | Version bump |
|-------------|-------------|
| `fix:` | Patch (1.0.x) |
| `feat:` | Minor (1.x.0) |
| `BREAKING CHANGE:` in commit body | Major (x.0.0) |
| `chore:`, `docs:`, `refactor:` | None |

GitHub Actions uses OIDC trusted publishing -- no `NPM_TOKEN` needed.
