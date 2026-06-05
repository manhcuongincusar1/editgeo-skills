# Recipes — common BĐS shots, end to end

This page gives ready patterns that combine camera + data + markers. Each names the DataAsset directive it needs (baked by `editgeo resolve` — see editgeo-data) and the slots to write.

## Highlight a plot / building
Trace the real footprint and label its size.
```json map.json
"assets": { "plot": { "id": "plot", "type": "geojson", "resolve": "building",
  "data": { "type": "FeatureCollection", "features": [ { "type": "Feature", "properties": {},
    "geometry": { "type": "Polygon", "coordinates": [[ /* rough rect around the address */ ]] } } ] } } },
"slots": [
  { "id": "r-plot", "type": "geojson", "dataAssetId": "plot", "drawingOptionId": "plotHi",
    "items": [ { "time": 2.2, "transitionIn": 0.7, "value": { "opacity": 1, "autoTransitionOpacity": true } } ] }
]
```
`resolve:"building"` snaps the rough rect to the real OSM footprint. Add a marker with the area (measurements.md). Orbit it with the camera recipe.

## Draw a route to an amenity (with ETA)
```json
"assets": { "route": { "id": "route", "type": "geojson", "snap": "driving",
  "data": { "type": "FeatureCollection", "features": [ { "type": "Feature", "properties": {},
    "geometry": { "type": "LineString", "coordinates": [ [PROP_LNG,PROP_LAT], [DEST_LNG,DEST_LAT] ] } } ] } } },
"slots": [
  { "id": "r-route", "type": "geojson", "dataAssetId": "route", "drawingOptionId": "routeGlow",
    "items": [ { "time": 3, "transitionIn": 0.95, "easer": "inOutQuart",
      "value": { "opacity": 1, "trimLineStart": 0, "trimLineEnd": 1 } } ] }
]
```
`snap:"driving"` replaces the straight line with the real road polyline (OSRM) and exposes duration/distance for the ETA label. `trimLineEnd` × the keyframe progress draws the line on. Fit it with `fitFeature` (camera.md).

## Journey through several stops (e.g. a metro line)
Camera moves station → station; each station gets a 2-line info box (markers.md).
- One `viewport` with a keyframe pair per station (arrive, hold) — keep holds short.
- One `marker` slot per station: `text` = name, `subtitle` = a one-line fact, timed to the camera arrival.
- Optional landmarks: extra markers; a key site can get a `resolve:"building"` polygon or a `boundary` highlight.
See the metro showcase pattern; pace it fast and decisive.

## Highlight a ward (phường) or named road
Use the resolver's `query` directive (editgeo-data) — **do not** hand-draw the boundary.
```json
"assets": {
  "ward": { "id": "ward", "type": "geojson", "query": { "kind": "boundary", "name": "Phường Bến Thành", "near": [106.698, 10.772] } },
  "mainRd": { "id": "mainRd", "type": "geojson", "query": { "kind": "road", "name": "Lê Lợi", "near": [106.700, 10.773] } }
}
```
Boundary → filled polygon with a light-running border; road → a highlighted LineString. VN wards (post-2025 reform) replace districts — see editgeo-data for current naming and `near` rules.

## Show nearby POIs
```json
"assets": { "schools": { "id": "schools", "type": "geojson",
  "query": { "kind": "poi", "category": "school", "near": [106.70, 10.776], "radius": 1500, "limit": 6 } } }
```
The points auto-become markers (markers.md). Combine with routes to show "X minutes to school/market/hospital".

## Subdivide land into lots (phân lô) — the staggered reveal
For a "phân lô bán nền" project, draw each lot as its **own** `geojson` slot (so it gets its own border + reveal time), and number them with one `marker` slot. Zoom in close so the grid fills frame.
- One DataAsset per lot (a small Polygon), one `geojson` slot each; stagger `items[].time` (e.g. `13 + i*0.55`) for a one-by-one draw-on.
- Color-code by sharing two drawingOptions: e.g. `lotAvail` (gold) vs `lotSold` (gray).
- Lot numbers: a single `marker` slot whose `items[]` are the per-lot labels (`text:"1".."N"`, centered on each lot, staggered).
- Wrap the block in a master-parcel Polygon + spotlight (`media` `clipMode:"mask"`, media.md) and a glowing border (stroke + `trimLineEnd`).
- A builder script that loops the grid keeps the IR small. Add `extrusion` on a few lots for 3D "nhà mẫu".

## Clip satellite/render imagery to a parcel (masked-imagery hero)
A `media` image clipped to the plot shape, real basemap kept around it, glowing border — the premium "here's the land" shot. See [media.md](media.md) (`clipMode:"clip"` keeps the map outside; `"mask"` spotlights). Pair with the plot's own border slot revealing via `trimLineEnd`.

## Data over time (by year) / a moving trip
Earthquakes/sales/projects appearing year by year, or a delivery trip with a comet trail. Declare `DataAsset.time` (events or trip) + keyframe the `now` playhead on the slot — see [temporal.md](temporal.md). Use `pointMode:"circle"`/`"heatmap"` for many points.
