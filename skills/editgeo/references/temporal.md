# Temporal — data that animates over time (events by year, moving trips)

Show data unfolding through time: earthquakes/sales/projects by year, population growth, a delivery/GPS trip with a comet trail. Declared on a **DataAsset** via `time` (a `TimeConfig`) + a **`now` playhead** keyframed on the geojson slot. Deterministic: the engine injects one scalar `now` per frame.

## Two modes

### `events` — features appear/disappear by a timestamp
Each feature carries a timestamp in `properties[field]`. As `now` advances, features reveal (and optionally fade out).
```json
"quakes": { "id": "quakes", "type": "geojson",
  "time": { "mode": "events", "field": "year", "start": 1990, "end": 2024, "reveal": "cumulative", "fadeIn": 1.2 },
  "data": { "type": "FeatureCollection", "features": [
    { "type": "Feature", "geometry": { "type": "Point", "coordinates": [108.2,16.05] }, "properties": { "year": 1994, "mag": 6.1 } }
  ] } }
```
- `reveal`: `"cumulative"` (all `t ≤ now`, stay) | `"window"` (only `now-windowSize ≤ t ≤ now`, with `fadeOut`).
- `fadeIn` / `fadeOut` — fade span in DATA units. Pair with a `circle`/`heatmap` `pointMode` (layers-styling.md) so thousands of points scale; `radiusProp`/`weightProp` can map magnitude.

### `trip` — a moving object with a comet trail
The feature is a LineString whose `properties[field]` is a parallel array of per-vertex times.
```json
"route": { "id": "route", "type": "geojson",
  "time": { "mode": "trip", "field": "times", "start": 0, "end": 80, "trailLength": 22, "headMarkerSlotId": "head" },
  "data": { "type": "FeatureCollection", "features": [
    { "type": "Feature", "properties": { "times": [0,12,24,38,50,62,72,80] },
      "geometry": { "type": "LineString", "coordinates": [[108.214,16.060], "…8 points…"] } } ] } }
```
- `trailLength` — comet trail length in DATA units (the lit segment behind the head).
- `headMarkerSlotId` — id of a `marker` slot whose pin **rides the head** of the trip at `now`.

## The `now` playhead — map video-time → data-time
Keyframe `now` on the geojson slot's items (linear easer = constant speed):
```json
{ "id": "s-quakes", "type": "geojson", "dataAssetId": "quakes", "drawingOptionId": "qk", "items": [
  { "time": 0, "value": { "now": 1990 } },
  { "time": 8, "easer": "linear", "value": { "now": 2024 } } ] }
```
At each frame the engine interpolates `now` and reveals/animates accordingly. Same IR → same frame. (For a trip, keyframe `now` from `start`→`end`.)

## Notes
- A temporal slot needs **≥2 `now` keyframes** or it never advances.
- Times must be numeric (and for trips, monotonic, one per vertex). No resolver step — temporal is authored data, not baked.
