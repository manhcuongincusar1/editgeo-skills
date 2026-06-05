# Layer styling — drawingOptions (fill, stroke, dash, pattern, casing, 3D, circle, heatmap, passthrough)

A `geojson` slot binds a DataAsset (geometry) to a `drawingOption` (style) via `drawingOptionId`. This page is the full style vocabulary. Everything is declarative — if a look isn't typed below, use the **paint/layout passthrough** escape hatch (raw MapLibre, full expressions).

## fill / stroke (polygons & lines)
```json
"plotHi": { "id": "plotHi",
  "fill":   { "fillColor": { "r": 255, "g": 205, "b": 70, "a": 1 }, "opacity": 0.3 },
  "stroke": { "strokeColor": { "r": 255, "g": 220, "b": 110, "a": 1 }, "strokeWidth": 2.5 } }
```
- Polygon → animated fill + a light-running glow border. LineString → draw-on via `trimLineEnd`.
- Colors are `{r,g,b,a}` (0–255 except a 0–1).

## Dashed lines + marching ants
```json
"stroke": { "strokeColor": {…}, "strokeWidth": 4, "dashVal": 2, "dashSpeed": 8, "strokeGapColor": { "r": 20,"g":30,"b":50,"a":1 } }
```
- `dashVal` — dash/gap length in line-widths (>0 → dashed; **disables** the running-light glow).
- `dashSpeed` — >0 marches the ants (animated). `strokeGapColor` — fills the gaps (2-tone dash).

## Road casing (outlined line)
`stroke.strokeOutlineColor` (+ `strokeOutlineWidth`, default 2) draws a wider casing UNDER the stroke — the classic divided-road / highlighted-route look.

## Fill pattern (textured polygon)
```json
"images": { "hatch": { "svg": "<svg …>…</svg>", "width": 16, "height": 16 } },
"drawingOptions": { "zoneP": { "fill": { "patternType": "hatch" } } }
```
- `fill.patternType` = id of a doc-level **`images`** entry (an `ImageAsset`: inline `svg` or `url`/data-URI). Tiled across the polygon. Register the image once in `images{}`; reference by id.

## 3D extrusion (buildings / prisms that grow)
```json
"house": { "id": "house", "extrusion": 16, "fill": { "fillColor": {…}, "opacity": 0.9 }, "stroke": { … } }
```
- `extrusion` (meters) extrudes a polygon into a 3D prism; it **grows in** over the slot's `transitionIn`. Needs camera `pitch`. Great for "the building rises".

## Point layers — circle & heatmap (scale to many points)
A `geojson` slot whose geometry is Points renders as DOM markers by default. Set `pointMode` for GPU layers:
```json
"quakes": { "id": "quakes", "pointMode": "circle",
  "circle": { "radiusProp": "mag", "radius": 34, "color": { "r": 255,"g":80,"b":70,"a":1 }, "blur": 0.35, "strokeColor": {…}, "strokeWidth": 1.5 } }
"dens":   { "id": "dens", "pointMode": "heatmap",
  "heatmap": { "weightProp": "value", "radius": 55, "intensity": 1.1 } }
```
- `pointMode`: `"marker"` (default, DOM pins) | `"circle"` (GPU dots) | `"heatmap"` (density).
- `circle.radiusProp` / `heatmap.weightProp` — drive size/weight from a feature property (e.g. magnitude, value). Ideal for many data points (and temporal — see temporal.md).

## Passthrough — the declarative escape hatch
Any MapLibre paint/layout property (incl. full Style-Spec expressions) merged onto the layer AFTER the typed fields:
```json
"grad": { "id": "grad",
  "paint":  { "fill-color": ["interpolate", ["linear"], ["get", "price"], 0, "#1e3a8a", 100, "#f59e0b"] },
  "layout": { "line-cap": "round" } }
```
- `paint` / `layout` win over typed defaults. Use for data-driven color ramps, gradients, or any effect not yet typed above — the engine stays 100% declarative without per-video code.

## geojson slot value (per-keyframe, on the slot's items)
`opacity`, `size`, `trimLineStart`/`trimLineEnd` (0–1 line reveal), `lineOffset`, `autoTransitionOpacity`, `now` (temporal playhead — temporal.md), `deformField` (procedural wobble: `{active,type:"random"|"perlin",strengthX,strengthY,scaleX,scaleY}`).
