# Camera — fly-to, orbit, cinematic earth-dive, follow, fit

This page teaches the `viewport` slot: how the camera moves. One `viewport` slot per IR, with keyframed `items[]`. The house feel is **fast and decisive** — short holds, confident eases, no drifting.

## The viewport value
```json
{ "time": 0, "easer": "inOutCubic", "interpolator": "flyTo",
  "value": { "lng": 106.7009, "lat": 10.7769, "zoom": 16.5, "bearing": 0, "pitch": 56 } }
```
- `lng/lat` — center (or use `place` for the resolver to geocode — see editgeo-data).
- `zoom` 0–22 · `bearing` 0–360 (rotation) · `pitch` 0–85 (tilt; 3D buildings need pitch + `basemap.buildings3d`).
- Between two items each field interpolates; **bearing takes the SHORTEST arc**.

## Easers
`linear · inOutSine · inOutQuad · inOutCubic · inOutQuart · inOutQuint · inOutExpo · inOutCirc · inOutBack · inOutElastic`. Use `inOutCubic`/`inOutQuart` for confident moves, `linear` for constant-rate spins.

## Fly-to
Two keyframes: start wide, end on the subject. Keep it short (1.5–2.5s).
```json
{ "time": 0,   "easer": "inOutCubic", "value": { "lng": 106.70, "lat": 10.79, "zoom": 11, "pitch": 0 } },
{ "time": 2.2, "easer": "inOutQuart", "value": { "lng": 106.7009, "lat": 10.7769, "zoom": 17, "pitch": 55 } }
```

## Cinematic earth-dive (globe → subject)
Set `basemap.projection: "globe"`. Start near-space (zoom ~2.5), hold a beat, then dive in **ONE eased segment with a FIXED center** — multiple eased pan+zoom waypoints stutter at each stop.
```json
{ "time": 0,   "easer": "linear",     "value": { "lng": 106.70, "lat": 10.78, "zoom": 2.5, "pitch": 0 } },
{ "time": 1.4, "easer": "inOutCubic", "value": { "lng": 106.70, "lat": 10.78, "zoom": 2.8, "pitch": 0 } },
{ "time": 6.8, "easer": "inOutCubic", "value": { "lng": 106.7009, "lat": 10.7769, "zoom": 14.9, "pitch": 56 } }
```
Keep `lng/lat` constant through the dive (move the center only after you've arrived) so the descent reads as a straight plunge, not a swerve.

## Orbit a building (pure IR, no engine support)
Bearing interpolates by **shortest arc**, so a single `0 → 360` keyframe spins 0°. Split the circle into hops < 180° and use `easer: "linear"` with a FIXED center (the centroid you compute):
```json
{ "time": 0, "easer": "linear", "value": { "lng": 106.7009, "lat": 10.7769, "zoom": 17.5, "pitch": 60, "bearing": 0 } },
{ "time": 3, "easer": "linear", "value": { "...same lng/lat/zoom/pitch...", "bearing": 120 } },
{ "time": 6, "easer": "linear", "value": { "...", "bearing": 240 } },
{ "time": 9, "easer": "linear", "value": { "...", "bearing": 360 } }
```
Set `basemap.buildings3d: true` and `pitch ~60`. See `assets/examples/orbit.json`.

## Fit a feature in frame (fit-bounds)
Frame a route or plot to its extent — the engine computes the camera from the feature's bounds:
```json
{ "time": 5, "easer": "inOutQuart", "value": { "bearing": -10, "pitch": 48, "fitFeature": { "slotId": "r-route", "padding": 120 } } }
```
`slotId` = a `geojson` slot id; `padding` in px.

## Follow a route (camera rides the line)
```json
{ "time": 4, "value": { "zoom": 16, "pitch": 60,
  "followFeature": { "slotId": "r-route", "progress": 0.5, "rotateAlong": true, "rotateAlongOffset": 0 } } }
```
Animate `progress` 0→1 across keyframes to travel the line; `rotateAlong` turns the camera into the heading.
