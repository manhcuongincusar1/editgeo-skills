---
name: editgeo-data
description: The data layer for editgeo map videos — DataAsset directives that turn names and rough shapes into real geometry, baked into the IR by `editgeo resolve`. Covers geocode (`place`), route snap (`snap`), building footprint (`resolve:"building"`), named road and admin boundary (`query.kind:"road"|"boundary"`), nearby POIs (`query.kind:"poi"`), external GeoJSON (`url`), icon search (`icon:"search:…"`), plus Vietnam-specific geography (the 2025 admin reform — wards replaced districts — and sourcing landmark coordinates from OSM). Use when deciding how to get real coordinates/shapes into an IR, why a place resolves wrong, or how VN ward/road/boundary names work. For writing the IR around these assets see the editgeo skill; for running `resolve` see the editgeo-cli skill.
---

# editgeo data layer

Real geometry never gets hand-typed. You declare a **directive** on a DataAsset (or a `place` on a viewport/marker), and `editgeo resolve` bakes it into concrete `lng/lat` and GeoJSON **in the IR file**, before preview/render. After resolve the IR is the Single Source of Truth: no directives left, renders offline, same pixels every time. Resolvers route through the gated EditBĐS server when logged in (metered); direct providers are a local-dev fallback.

## The directives
| Directive | On | Becomes | Source |
|---|---|---|---|
| `place: "Bến Thành, HCMC"` | viewport / marker `value` | `lng`/`lat` | geocoder |
| `snap: "driving"` | DataAsset (LineString) | road polyline + `duration`/`distance` | OSRM route |
| `resolve: "building"` | DataAsset (Polygon) | real building footprint Polygon | OSM/Overpass |
| `query.kind: "road"` | DataAsset | named highway → stitched LineString | OSM/Overpass |
| `query.kind: "boundary"` | DataAsset | admin relation → Polygon ring | OSM/Overpass |
| `query.kind: "poi"` | DataAsset | nearby points → FeatureCollection | POI search |
| `url: "https://…"` | DataAsset | fetched GeoJSON, **merged** with inline `data` | any GeoJSON URL |
| `icon: "search:metro"` / `"mdi:train"` | marker `value` | inline `iconSvg` | Iconify |

### query shapes
```json
"query": { "kind": "poi", "category": "school", "near": [106.70, 10.776], "radius": 1500, "limit": 6 }
"query": { "kind": "road", "name": "Lê Lợi", "near": [106.700, 10.773], "radius": 800 }
"query": { "kind": "boundary", "name": "Phường Bến Thành", "near": [106.698, 10.772], "radius": 4000 }
```
- **`road`/`boundary` need `near` as `[lng,lat]` coordinates** — this resolver does not geocode `near`. Get the coordinate first (a `place` elsewhere, or a known landmark), then put numbers in `near`.
- `name` is matched case-insensitively as an OSM-name substring; it falls back to `category`.

### url alongside inline data
`url` does NOT replace inline `data` — it **merges**: the resolver appends the fetched features to whatever `data.features` you already wrote. Use `url` to keep big GeoJSON out of the IR (saves tokens) while still adding a few inline features.

## Resolve order matters
`resolve` geocodes `place` FIRST (so building/road/boundary/route/POI have coordinates), then footprints, roads, boundaries, routes, POIs, urls, and icons. Always `resolve` before `preview`/`render`; `render --strict` refuses an IR that still has directives.

## Vietnam geography
### 2025 admin reform — districts are gone
The provincial reform abolished **Quận/Huyện (districts)**. OSM administrative boundaries are now **Tỉnh/Thành phố (province) → Phường/Xã (ward)**. So:
- Highlight a **ward**, not a district: `"Phường Bến Thành"`, not `"Quận 1"`. Using an old district name returns no boundary.
- Use **current ward names** for `query.kind:"boundary"`.

### Sourcing landmark coordinates
Geocoders are unreliable for VN landmarks (they drift to the wrong district or another country). For named landmarks, prefer **OSM/Overpass** coordinates. Known-good HCMC anchors used in demos:
| Place | `[lng, lat]` |
|---|---|
| Bến Thành (center Q.1) | `[106.698, 10.772]` |
| Landmark 81 | `[106.7219, 10.7951]` |
| Vinhomes Grand Park (Thủ Đức) | `[106.8417, 10.8398]` |
| Tân Sơn Nhất airport | `[106.6557, 10.8181]` |
| Bến xe Miền Đông Mới | `[106.8162, 10.8796]` |

When a `place` resolves to an obviously wrong spot, replace it with a verified `lng/lat` (sanity-check against the map) rather than trusting the geocode.

## After resolve
The IR holds only concrete geometry. If you change a directive (new place, new route), re-run `resolve`. The resolver layer is the product's gated IP — treat the server as the source for real map data when logged in.
