# FX, world & basemap — the cinematic finish

This page teaches the `fx` slot (color/look), the `world` slot (terrain/fog/light), and the `basemap` block (tiles/style/projection). These set the mood; keep them tasteful — the EditBĐS look is clean and confident, not over-graded.

## basemap
```json
"basemap": { "styleTemplate": "satellite", "vars": { "tiles": "{maptiler}" }, "projection": "globe", "buildings3d": true }
```
- `styleTemplate` — `satellite` (aerial), `vector` (streets), `white` (clean light — good for label-forward listings), `dark`.
- `projection` — `globe` (needed for the earth-dive, camera.md) or `mercator` (flat).
- `buildings3d` — extrude 3D buildings (needs camera `pitch`).

## world slot (terrain / fog / light)
One set of values (first item):
```json
{ "id": "world", "type": "world", "items": [ { "time": 0, "value": {
  "elevation": 1.1, "fogDensity": 0.05,
  "lightIntensity": 0.5, "lightPositionR": 1.2, "lightPositionA": 210, "lightPositionP": 35 } } ] }
```
- `elevation` — terrain exaggeration. `fogDensity` — atmospheric haze.
- `light*` — sun: intensity, radial/azimuth/polar position (controls building shadows).

## fx slot (grade / vignette / tilt-shift)
Keyframe the look — e.g. punch up saturation/contrast as the camera arrives:
```json
{ "id": "grade", "type": "fx", "items": [
  { "time": 0,   "easer": "inOutQuad", "value": { "saturation": 1.0, "contrast": 1.0, "brightness": 1.0,
    "vignette": { "strength": 0.12, "size": 0.95, "softness": 0.6 }, "tiltShift": { "blur": 0, "blurMargin": 0.5, "focusArea": 0.34 } } },
  { "time": 2.6, "easer": "inOutQuad", "value": { "saturation": 1.3, "contrast": 1.16, "brightness": 1.04,
    "vignette": { "strength": 0.5, "size": 0.7, "softness": 0.5 }, "tiltShift": { "blur": 5, "blurMargin": 0.34, "focusArea": 0.3 } } }
] }
```
- Color: `saturation`, `contrast`, `brightness`, `hue`, `grayscale`, `sepia`, `blur`.
- `vignette` `{ strength, size, softness }` — darken the edges for focus.
- `tiltShift` `{ blur, blurMargin, focusArea }` — fake-miniature; great over 3D-building dives. (Disabled in capture mode automatically where it would break.)

Keep grades subtle at the wide shot and intensify on arrival — it reads as "the camera found its subject".
