# Recipes — common BĐS shots, end to end

This page gives ready patterns that combine camera + data + markers. Each names the DataAsset directive it needs (baked by `editgeo resolve` — see editgeo-data) and the slots to write. Pair with the example IRs in `assets/examples/`.

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
