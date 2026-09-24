# AGENTS.md: rendezvous-kit

Instructions for AI coding agents working in this repository.

## What This Is

TypeScript library for finding fair meeting points for N people. Pipeline: isochrone computation, polygon intersection, venue search (Overpass API), route matrix, fairness scoring. Only runtime dependency is `geohash-kit` (ours); otherwise zero third-party dependencies.

## Repository Layout

```
src/
  types.ts             : shared TypeScript interfaces (RoutingEngine, LatLon, FairnessStrategy, etc.)
  geo.ts               : pure-TypeScript polygon geometry (Sutherland-Hodgman intersection, area, bbox, centroid, circle)
  hull.ts               : convex hull fast-path strategy (fast path for closely spaced participants)
  engines/
    valhalla.ts         : Valhalla adapter (isochrone + matrix + route with turn-by-turn)
    openrouteservice.ts : ORS adapter (isochrone + matrix; computeRoute not yet implemented)
    graphhopper.ts      : GraphHopper adapter (isochrone + matrix; computeRoute not yet implemented)
    osrm.ts              : OSRM adapter (matrix only, no isochrone or route)
  venues.ts             : Overpass API venue search with tag mapping
  rendezvous.ts         : findRendezvous pipeline (the main function)
  validate.ts           : input validation helpers (URL, timeout, JSON parsing)
  index.ts              : barrel re-export
examples/               : runnable TypeScript examples
docs/                   : guides and interactive demo page (GitHub Pages)
```

## Commands

```bash
npm install         # install dependencies
npm run build       # compile TypeScript -> dist/
npm test            # run all tests (vitest)
npm run test:watch  # vitest watch mode
npm run typecheck   # type-check without emitting
npm run bench       # performance benchmarks
```

There is no separate lint script.

## Subpath Exports

| Import path | Module |
|-------------|--------|
| `rendezvous-kit` | main barrel: all public exports |
| `rendezvous-kit/geo` | polygon geometry utilities |
| `rendezvous-kit/venues` | Overpass venue search |
| `rendezvous-kit/rendezvous` | `findRendezvous` pipeline function |
| `rendezvous-kit/engines/valhalla` | `ValhallaEngine` |
| `rendezvous-kit/engines/openrouteservice` | `OpenRouteServiceEngine` |
| `rendezvous-kit/engines/graphhopper` | `GraphHopperEngine` |
| `rendezvous-kit/engines/osrm` | `OsrmEngine` |

## Engine Configuration

All engines are configured programmatically (no env vars): pass URLs and keys to constructors.

| Engine | Required config | Notes |
|--------|----------------|-------|
| `ValhallaEngine` | `baseUrl` | Self-hosted or an L402-gated routing service |
| `OpenRouteServiceEngine` | `apiKey` | Free key from openrouteservice.org/dev; `baseUrl` optional override |
| `GraphHopperEngine` | `baseUrl` | `apiKey` optional (needed for hosted GraphHopper) |
| `OsrmEngine` | `baseUrl` | Matrix only: no isochrone support |

Overpass venue search uses public endpoints by default. Pass `overpassUrl` to `searchVenues()` to override.

## Key Interfaces

- `RoutingEngine`: implement `computeIsochrone`, `computeRouteMatrix`, `computeRoute` to add a new engine
- `RendezvousOptions`: input to `findRendezvous` (participants, mode, time, venues, fairness)
- `RendezvousSuggestion`: output venue, travel times per participant, fairness score

## Conventions

- British English everywhere: favour, colour, behaviour, licence, initialise, metre
- Only dependency: `geohash-kit` (ours): uses its `pointInPolygon`, GeoJSON types, distance utilities
- Git: commit messages use `type: description` format (e.g. `feat:`, `fix:`, `docs:`). Do not include `Co-Authored-By` lines
- TDD: write a failing test first, then implement
- ESM-only, with `.js` extensions in imports even for `.ts` source files
- TypeScript strict mode: no `any`, no implicit returns
- GeoJSON coordinates: `[longitude, latitude]`, never `[lat, lon]`
- JSDoc on all public exports
- No third-party runtime dependencies

## Testing Strategy

Tests are co-located with source files (`*.test.ts`). Engine tests mock HTTP responses via `globalThis.fetch`: no real HTTP calls are made in tests. `rendezvous.test.ts` tests the full pipeline using mock engines, not real routing services. `hull.ts` is a convex hull fast-path strategy: an optimisation, not a replacement for full isochrone intersection.

## Common Pitfalls

- `findRendezvous` requires at least 2 participants: single-origin use cases should query the engine directly
- OSRM cannot compute isochrones: calling `findRendezvous` with an `OsrmEngine` fails unless `strategy: 'hull'` is used
- The Overpass API has rate limits: in production, configure a self-hosted Overpass instance via `overpassUrl`
- Valhalla returns distance in km; ORS returns metres: the engine adapters normalise this
- Coordinate order is `[lon, lat]` throughout, matching GeoJSON but opposite to what many mapping APIs expect: check carefully at engine adapter boundaries

## Before Committing

1. Run `npm test && npm run typecheck`: both must pass
2. Use conventional commit messages: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`
3. Do not add third-party runtime dependencies
4. Use British English in all text
5. Add JSDoc to any new public exports

## Release & Versioning

Automated via [forgesworn/anvil](https://github.com/forgesworn/anvil): `auto-release.yml` reads conventional commits on push to `main`, bumps the version, and creates a GitHub Release; `release.yml` then runs the pre-publish gates and publishes to npm via OIDC trusted publishing.

| Commit type | Version bump |
|-------------|-------------|
| `fix:` | Patch (1.0.x) |
| `feat:` | Minor (1.x.0) |
| `BREAKING CHANGE:` (in commit body) | Major (x.0.0) |
| `chore:`, `docs:`, `refactor:` | None |

Tests must pass before release. Work on feature branches; merge to main only when a logical chunk is complete to avoid version spam.
