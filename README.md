# rendezvous-kit

**Nostr:** [`npub1mgvlrnf5hm9yf0n5mf9nqmvarhvxkc6remu5ec3vf8r0txqkuk7su0e7q2`](https://njump.me/npub1mgvlrnf5hm9yf0n5mf9nqmvarhvxkc6remu5ec3vf8r0txqkuk7su0e7q2)

**Find fair meeting points for N people — isochrone intersection, venue search, and fairness scoring.**

[![npm](https://img.shields.io/npm/v/rendezvous-kit)](https://www.npmjs.com/package/rendezvous-kit)
[![licence](https://img.shields.io/npm/l/rendezvous-kit)](https://github.com/forgesworn/rendezvous-kit/blob/main/LICENSE)
![TypeScript](https://img.shields.io/badge/TypeScript-native-blue)
[![Nostr](https://img.shields.io/badge/Nostr-Zap%20me-purple)](https://primal.net/p/npub1mgvlrnf5hm9yf0n5mf9nqmvarhvxkc6remu5ec3vf8r0txqkuk7su0e7q2)
[![GitHub Sponsors](https://img.shields.io/github/sponsors/TheCryptoDonkey?logo=githubsponsors&color=ea4aaa&label=Sponsor)](https://github.com/sponsors/TheCryptoDonkey)

**[Live Demo →](https://forgesworn.github.io/rendezvous-kit)**

### How it works

```
  Alice (London)        Bob (Bristol)        Carol (Birmingham)
       ╲                     │                     ╱
    isochrone            isochrone            isochrone
       ╲                     │                     ╱
        ╲────────── intersection ──────────╱
                         │
                   venue search
                         │
                  fairness scoring
                         │
                 ranked suggestions
```

Geographic midpoint tools find the centre of a triangle. **rendezvous-kit finds where everyone can actually get to** — using real road networks, travel times, and venue availability.

## Why rendezvous-kit?

- **Full pipeline** — isochrone computation → polygon intersection → venue search → fairness scoring, all in one library
- **Engine-agnostic** — bring your own routing engine: Valhalla, OpenRouteService, GraphHopper, or OSRM
- **Fairness strategies** — `min_max` (minimise worst case), `min_total` (minimise sum), `min_variance` (equalise travel times)
- **Built on geohash-kit** — leverages our spatial primitives; only runtime dependency
- **Zero third-party dependencies** — ships with a pure-TypeScript Sutherland–Hodgman polygon intersection, Overpass API venue search, and all engine adapters

## Use Cases

- **Social apps** — "Meet in the middle" features for groups of friends picking a pub, café, or restaurant
- **Ride-sharing** — optimal pickup points that minimise detour for all passengers
- **Event planning** — conference venues fair to attendees from multiple cities
- **Corporate** — office locations that balance commute times across a distributed team
- **Delivery logistics** — hub placement that minimises worst-case last-mile distances
- **Emergency services** — staging areas reachable by multiple response units within a time budget

## Compared to Alternatives

| Capability | Geographic midpoint libraries | Commercial APIs (Targomo, TravelTime) | rendezvous-kit |
|---|---|---|---|
| Travel-time aware | No — straight-line distance only | Yes | Yes |
| Multi-person intersection | No | Yes (API key + billing) | Yes (self-hosted, free) |
| Fairness weighting | No | No | Yes — 3 strategies |
| Venue search included | No | No | Yes — via Overpass API |
| Engine-agnostic | N/A | No — vendor lock-in | Yes — 4 engines |
| Works offline/self-hosted | N/A | No | Yes (with local Valhalla/OSRM) |

rendezvous-kit is the only open-source library that combines isochrone intersection, venue discovery, and fairness scoring in a single pipeline.

## Install

```bash
npm install rendezvous-kit
```

## Quick Start

```typescript
import { findRendezvous } from 'rendezvous-kit'
import { ValhallaEngine } from 'rendezvous-kit/engines/valhalla'

const engine = new ValhallaEngine({ baseUrl: 'http://localhost:8002' })

const suggestions = await findRendezvous(engine, {
  participants: [
    { lat: 51.5074, lon: -0.1278, label: 'Alice' },  // London
    { lat: 51.4545, lon: -2.5879, label: 'Bob' },    // Bristol
    { lat: 52.4862, lon: -1.8904, label: 'Carol' },  // Birmingham
  ],
  mode: 'drive',
  maxTimeMinutes: 90,
  venueTypes: ['cafe', 'restaurant'],
  fairness: 'min_max',
  limit: 5,
})

for (const s of suggestions) {
  console.log(`${s.venue.name} — score: ${s.fairnessScore.toFixed(1)} min`)
  console.log('  Travel times:', s.travelTimes)
}
```

## Engine Support

| Engine | Isochrone | Route Matrix | Route | Auth |
|--------|:---------:|:------------:|:-----:|------|
| Valhalla | Yes | Yes | Yes | None (self-hosted) |
| OpenRouteService | Yes | Yes | No | API key |
| GraphHopper | Yes | Yes | No | API key (optional) |
| OSRM | No | Yes | No | None (self-hosted) |

OSRM does not support isochrone computation — use it only when you need a fast route matrix and are supplying your own intersection polygon.

## Fairness Strategies

| Strategy | Optimises | Use when... |
|----------|-----------|-------------|
| `min_max` | Worst-case travel time | You want nobody to travel excessively |
| `min_total` | Sum of all travel times | You want minimum total travel for the group |
| `min_variance` | Variance in travel times | You want everyone to travel roughly equally |

## API Reference

### Core function

| Function | Description |
|----------|-------------|
| `findRendezvous(engine, options)` | Run the full pipeline and return ranked suggestions |

### Geometry (`rendezvous-kit/geo`)

| Function | Description |
|----------|-------------|
| `intersectPolygons(polygons)` | Sutherland–Hodgman N-polygon intersection; returns largest component or `null` |
| `intersectPolygonsAll(polygons)` | N-polygon intersection preserving all disconnected components; returns `GeoJSONPolygon[]` |
| `boundingBox(polygon)` | Compute `BBox` (minLon, minLat, maxLon, maxLat) |
| `centroid(polygon)` | Geometric centre as `{ lat, lon }` |
| `polygonArea(polygon)` | Area in square metres |
| `circleToPolygon(centre, radiusMetres, segments?)` | Approximate a circle as a GeoJSON Polygon (default 64 segments) |
| `getDestinationPoint(start, distanceMetres, bearingDeg)` | Haversine destination point from `[lon, lat]`, distance, and bearing |

### Engines

| Class | Import path | Constructor |
|-------|-------------|-------------|
| `ValhallaEngine` | `rendezvous-kit/engines/valhalla` | `{ baseUrl }` |
| `OpenRouteServiceEngine` | `rendezvous-kit/engines/openrouteservice` | `{ apiKey, baseUrl? }` |
| `GraphHopperEngine` | `rendezvous-kit/engines/graphhopper` | `{ baseUrl, apiKey? }` |
| `OsrmEngine` | `rendezvous-kit/engines/osrm` | `{ baseUrl }` |

### Venues (`rendezvous-kit/venues`)

| Function | Description |
|----------|-------------|
| `searchVenues(polygon, venueTypes, overpassUrl?)` | Search Overpass API within polygon bounding box |

### Types

| Type | Shape |
|------|-------|
| `LatLon` | `{ lat, lon, label? }` |
| `GeoJSONPolygon` | Standard GeoJSON polygon geometry |
| `TransportMode` | `'drive' \| 'cycle' \| 'walk' \| 'public_transit'` |
| `FairnessStrategy` | `'min_max' \| 'min_total' \| 'min_variance'` |
| `VenueType` | `'park' \| 'cafe' \| 'restaurant' \| 'service_station' \| 'library' \| 'pub' \| 'playground' \| 'community_centre' \| 'bar' \| 'fast_food' \| 'garden' \| 'theatre' \| 'arts_centre' \| 'fitness_centre' \| 'sports_centre' \| 'escape_game' \| 'swimming_pool' \| string` |
| `RoutingEngine` | Interface — `computeIsochrone` + `computeRouteMatrix` + `computeRoute` |
| `Isochrone` | `{ origin, mode, timeMinutes, polygon }` |
| `MatrixEntry` | `{ originIndex, destinationIndex, durationMinutes, distanceKm }` |
| `RouteMatrix` | `{ origins, destinations, entries }` |
| `Venue` | `{ name, lat, lon, venueType, osmId? }` |
| `RendezvousOptions` | `{ participants, mode, maxTimeMinutes, venueTypes, fairness?, limit?, strategy? }` |
| `RendezvousSuggestion` | `{ venue, travelTimes, fairnessScore, metadata? }` |
| `RouteGeometry` | `{ origin, destination, mode, durationMinutes, distanceKm, geometry, legs? }` |
| `RouteLeg` | `{ instruction, distanceKm, durationMinutes, type?, streetNames?, ... }` |
| `GeoJSONLineString` | `{ type: 'LineString', coordinates: number[][] }` |
| `BBox` | `{ minLon, minLat, maxLon, maxLat }` |
| `Coordinate` | `{ lat, lon }` |

## Subpath Exports

```typescript
import { findRendezvous } from 'rendezvous-kit'                          // barrel
import { intersectPolygons, intersectPolygonsAll, centroid,
         circleToPolygon, getDestinationPoint } from 'rendezvous-kit/geo' // geometry
import { ValhallaEngine } from 'rendezvous-kit/engines/valhalla'
import { OpenRouteServiceEngine } from 'rendezvous-kit/engines/openrouteservice'
import { GraphHopperEngine } from 'rendezvous-kit/engines/graphhopper'
import { OsrmEngine } from 'rendezvous-kit/engines/osrm'
import { searchVenues } from 'rendezvous-kit/venues'
import { findRendezvous } from 'rendezvous-kit/rendezvous'               // same as barrel
```

## Pipeline

`findRendezvous` runs six steps:

1. **Isochrones** — compute a reachability polygon for each participant
2. **Intersection** — intersect all polygons using Sutherland–Hodgman (supports concave shapes and disconnected components)
3. **Venue search** — query Overpass API within the intersection's bounding box
4. **Route matrix** — compute travel times from every participant to every candidate venue
5. **Filtering** — remove venues where any participant exceeds the time budget or is unreachable
6. **Scoring** — apply the fairness strategy to produce a single score per venue
7. **Ranking** — sort by score ascending and return the top `limit` suggestions

If the isochrones do not overlap, `findRendezvous` returns an empty array. If no venues are found, it falls back to the geometric centroid of the intersection.

## Implementing a Custom Engine

```typescript
import type { RoutingEngine, LatLon, TransportMode, Isochrone, RouteMatrix, RouteGeometry } from 'rendezvous-kit'

class MyEngine implements RoutingEngine {
  readonly name = 'MyEngine'

  async computeIsochrone(origin: LatLon, mode: TransportMode, timeMinutes: number): Promise<Isochrone> {
    // call your API and return an Isochrone
  }

  async computeRouteMatrix(origins: LatLon[], destinations: LatLon[], mode: TransportMode): Promise<RouteMatrix> {
    // call your API and return a RouteMatrix
  }

  async computeRoute(origin: LatLon, destination: LatLon, mode: TransportMode): Promise<RouteGeometry> {
    // call your API and return a RouteGeometry with optional turn-by-turn legs
  }
}
```

## Examples

Runnable examples in [`examples/`](./examples/):

- **[basic-usage.ts](./examples/basic-usage.ts)** — find a fair meeting point for three people
- **[comparing-fairness-strategies.ts](./examples/comparing-fairness-strategies.ts)** — see how min_max, min_total, and min_variance rank differently
- **[custom-engine.ts](./examples/custom-engine.ts)** — implement the RoutingEngine interface with a mock engine

Run any example with `npx tsx examples/<name>.ts`.

## Guides

- **[Choosing a Fairness Strategy](./docs/choosing-a-fairness-strategy.md)** — when to use min_max vs min_total vs min_variance
- **[Self-Hosting a Routing Engine](./docs/self-hosting-a-routing-engine.md)** — run Valhalla, OSRM, or GraphHopper locally with Docker

## Companion Library

**geohash-kit** — spatial primitives (pointInPolygon, GeoJSON types, distance utilities) used internally by rendezvous-kit.

- npm: [`geohash-kit`](https://www.npmjs.com/package/geohash-kit)
- GitHub: [`forgesworn/geohash-kit`](https://github.com/forgesworn/geohash-kit)

## For AI Assistants

See [llms.txt](./llms.txt) for a concise API summary, or [llms-full.txt](./llms-full.txt) for the complete reference with examples.

## How the Pipeline Works

### Step 1: Parallel isochrone computation

`findRendezvous` computes a reachability polygon for each participant in parallel.
Every participant gets their own isochrone — a polygon representing everywhere they
can reach within `maxTimeMinutes` using the specified transport mode:

```typescript
// Internally, all N isochrones are fetched concurrently:
const isochrones = await Promise.all(
  participants.map(p => engine.computeIsochrone(p, mode, maxTimeMinutes))
)
```

This works identically for 2, 5, or 20 participants — each gets their own polygon.
The engine does the heavy lifting; rendezvous-kit just orchestrates the parallel calls.

### Step 2: N-polygon intersection

All isochrone polygons are intersected left-to-right using Sutherland–Hodgman clipping.
The result is the geographic area reachable by **every** participant within the time budget:

```typescript
import { intersectPolygonsAll } from 'rendezvous-kit/geo'

// Folds left-to-right: clip polygon 2 against 1, then clip 3 against that result, etc.
// Preserves disconnected components (e.g. two separate road corridors)
const components = intersectPolygonsAll(isochrones.map(iso => iso.polygon))
```

If the intersection is empty (participants too far apart), `findRendezvous` returns `[]`.

### Step 3: Venue search and scoring

Venues are searched within the intersection zone via Overpass API, then each venue is
scored using the chosen fairness strategy:

```typescript
const suggestions = await findRendezvous(engine, {
  participants: [alice, bob, carol],
  mode: 'drive',
  maxTimeMinutes: 90,
  venueTypes: ['cafe', 'restaurant'],
  fairness: 'min_max',   // minimise the worst individual travel time
  limit: 5,
})
// Each suggestion has: venue, travelTimes (per participant), fairnessScore
```

### When isochrones don't overlap

If the isochrones don't intersect, `findRendezvous` returns an empty array. Options:

1. **Increase `maxTimeMinutes`** — expand the reachability polygons
2. **Switch transport mode** — `'drive'` covers more ground than `'walk'`
3. **Use the `hull` strategy** — for nearby participants, the library automatically uses
   a convex hull of participant positions as the search region instead of isochrones.
   This always produces results because it searches the area *between* participants.

If the intersection exists but contains no matching venues, the library falls back to
a synthetic "Meeting point" at the area-weighted centroid of the intersection.

## Switching Routing Engines

OSRM does not support isochrone computation. If you're using OSRM and need the full
pipeline, swap to an isochrone-capable engine:

```typescript
// Before: OSRM (route matrix only — no isochrones)
import { OsrmEngine } from 'rendezvous-kit/engines/osrm'
const engine = new OsrmEngine({ baseUrl: 'http://localhost:5000' })

// After: Valhalla (self-hosted, free, full isochrone support)
import { ValhallaEngine } from 'rendezvous-kit/engines/valhalla'
const engine = new ValhallaEngine({ baseUrl: 'http://localhost:8002' })

// Or: OpenRouteService (hosted API, requires free API key)
import { OpenRouteServiceEngine } from 'rendezvous-kit/engines/openrouteservice'
const engine = new OpenRouteServiceEngine({
  apiKey: process.env.ORS_API_KEY!,
  baseUrl: 'https://api.openrouteservice.org', // default
})

// Or: GraphHopper (self-hosted or hosted, API key optional for self-hosted)
import { GraphHopperEngine } from 'rendezvous-kit/engines/graphhopper'
const engine = new GraphHopperEngine({
  baseUrl: 'http://localhost:8989',
  apiKey: process.env.GRAPHHOPPER_KEY, // optional for self-hosted
})

// All engines implement the same RoutingEngine interface.
// The rest of your code stays identical:
const suggestions = await findRendezvous(engine, { participants, mode, ... })
```

For self-hosting guides (Docker one-liners for Valhalla, OSRM, GraphHopper), see
[Self-Hosting a Routing Engine](./docs/self-hosting-a-routing-engine.md).

## Architecture

rendezvous-kit's pipeline combines two kinds of spatial operations:

**Routing engine operations** (external): isochrone computation, route matrix, and
route geometry. These call your chosen engine's HTTP API for real road-network
calculations.

**Local geometry operations** (pure TypeScript, no dependencies): polygon intersection
(Sutherland–Hodgman), area calculation (shoelace), centroid computation, bounding boxes,
and convex hull (via `geohash-kit/coverage`). These run entirely in-process with no
network calls.

The only external dependency is `geohash-kit`, which provides the `convexHull` function
used by the hull-strategy fast path for nearby participants. All other polygon operations
(intersection, clipping, triangulation, area) are implemented in `rendezvous-kit/geo`.

```
Participants ──→ Engine.computeIsochrone() ──→ Polygon[]
                                                  │
                                    intersectPolygonsAll()  ← local geometry
                                                  │
                                          Intersection zone
                                                  │
                                    searchVenues() via Overpass API
                                                  │
                              Engine.computeRouteMatrix() ──→ travel times
                                                  │
                                         scoreVenues()  ← local fairness calc
                                                  │
                                        Ranked suggestions
```

## Troubleshooting

**`findRendezvous` returns an empty array**
The participants' isochrones don't overlap — they are too far apart for the given `maxTimeMinutes`. Try increasing the time budget or switching to a faster transport mode.

**No venues found (fallback to centroid)**
The Overpass API found no matching venues within the intersection zone. This can happen in rural areas. The library returns a synthetic "Meeting point" at the area-weighted centroid as a fallback. Try broader `venueTypes` or a larger `maxTimeMinutes`.

**`RangeError: findRendezvous requires at least 2 participants`**
You must pass at least 2 participants. Single-origin use cases don't need a meeting-point library — query the routing engine directly.

**Engine HTTP errors (e.g., `Valhalla isochrone error: 400`)**
Check that your engine base URL is correct and the service is running. For ORS, verify your API key is valid. For Valhalla/OSRM, ensure the server has routing tiles loaded for the region you're querying.

**OSRM: `Error: OSRM does not support isochrone computation`**
OSRM cannot generate isochrones. Use Valhalla, OpenRouteService, or GraphHopper instead. OSRM is supported only for route matrix computation.

## Part of the ForgeSworn Toolkit

[ForgeSworn](https://forgesworn.dev) builds open-source cryptographic identity, payments, and coordination tools for Nostr.

| Library | What it does |
|---------|-------------|
| [nsec-tree](https://github.com/forgesworn/nsec-tree) | Deterministic sub-identity derivation |
| [ring-sig](https://github.com/forgesworn/ring-sig) | SAG/LSAG ring signatures on secp256k1 |
| [range-proof](https://github.com/forgesworn/range-proof) | Pedersen commitment range proofs |
| [canary-kit](https://github.com/forgesworn/canary-kit) | Coercion-resistant spoken verification |
| [spoken-token](https://github.com/forgesworn/spoken-token) | Human-speakable verification tokens |
| [toll-booth](https://github.com/forgesworn/toll-booth) | L402 payment middleware |
| [geohash-kit](https://github.com/forgesworn/geohash-kit) | Geohash toolkit with polygon coverage |
| [nostr-attestations](https://github.com/forgesworn/nostr-attestations) | NIP-VA verifiable attestations |
| [dominion](https://github.com/forgesworn/dominion) | Epoch-based encrypted access control |
| [nostr-veil](https://github.com/forgesworn/nostr-veil) | Privacy-preserving Web of Trust |

## Licence

[MIT](https://github.com/forgesworn/rendezvous-kit/blob/main/LICENSE)

## Support

For issues and feature requests, see [GitHub Issues](https://github.com/forgesworn/rendezvous-kit/issues).

If you find rendezvous-kit useful, consider sending a tip:

- **Lightning:** `thedonkey@strike.me`
- **Nostr zaps:** `npub1mgvlrnf5hm9yf0n5mf9nqmvarhvxkc6remu5ec3vf8r0txqkuk7su0e7q2`
