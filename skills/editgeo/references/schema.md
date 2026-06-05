# IR schema — the shape of a map video

This page is the field reference for the IR JSON. The authoritative contract is `schema/map-ir.schema.json` (+ typed `schema/map-ir.ts`); validate with `editgeo` (the CLI runs `validate`). Everything geo-anchored lives here; screen-anchored overlays (title/price/captions) live in the HyperFrames layer — see the editgeo-export skill.

## Top level
```json map.json
{
  "version": 1,
  "output": { "width": 1920, "height": 1080, "fps": 30, "duration": 8, "ss": 2 },
  "basemap": { "styleTemplate": "satellite", "vars": { "tiles": "{maptiler}" }, "projection": "globe", "buildings3d": true },
  "assets": { "<id>": { "...DataAsset..." } },
  "drawingOptions": { "<id>": { "...style..." } },
  "markerTemplates": { "<id>": { "...label style..." } },
  "images": { "<id>": { "svg": "<svg…>", "width": 16, "height": 16 } },
  "media": { "<id>": { "kind": "image", "url": "…", "coordinates": [ "…4 corners…" ] } },
  "slots": [ { "...timed layer..." } ]
}
```
- **output** — `width`/`height`/`fps`/`duration` (seconds) + `ss` (supersample, render only).
- **basemap** — `styleTemplate`: `satellite` | `vector` | `white` | `dark`; `projection`: `globe` | `mercator`; `buildings3d`; `vars.terrainTiles` for terrain. See [fx-world.md](fx-world.md).
- **images{}** — bitmaps/SVGs registered for `fill.patternType` / icons ([layers-styling.md](layers-styling.md)). **media{}** — georef image/video painted on the ground ([media.md](media.md)).

## slots[] — timed layers
Each slot has a `type` and (for animated ones) `items[]` of keyframes. Types:
| type | what it is |
|---|---|
| `viewport` | the camera (one per IR). [camera.md](camera.md) |
| `world` | terrain / fog / hillshade / light. [fx-world.md](fx-world.md) |
| `geojson` | a drawn layer (polygon/line/points) — DataAsset + drawingOption. [layers-styling.md](layers-styling.md) |
| `marker` | on-map labels/icons/rich cards. [markers.md](markers.md) |
| `media` | georef image/video on the ground (+ clip-to-shape). [media.md](media.md) |
| `fx` | color grade / vignette / tilt-shift. [fx-world.md](fx-world.md) |

**Keyframe item** (viewport/geojson/marker/fx):
```json
{ "time": 1.4, "easer": "inOutCubic", "value": { "...": "..." },
  "transitionIn": 0.6, "transitionOut": 0.4, "duration": 3 }
```
- `time` — seconds. `easer` — interpolation between this item and the next (see camera.md for the list).
- Between two items, every numeric field in `value` is interpolated; angles take the shortest arc.
- `transitionIn`/`Out` fade a layer in/out; `duration` bounds a marker's life.

## geojson slot
```json
{ "id": "r-plot", "type": "geojson", "dataAssetId": "plot", "drawingOptionId": "plotHi",
  "items": [ { "time": 2, "transitionIn": 0.7, "value": { "opacity": 1, "autoTransitionOpacity": true, "trimLineStart": 0, "trimLineEnd": 1 } } ] }
```
- Polygon → animated fill + light-running border. LineString → draw-on via `trimLineEnd` × progress. Points → auto POI markers (or GPU `circle`/`heatmap` via `drawingOption.pointMode`).
- `value` also carries `now` (temporal playhead — [temporal.md](temporal.md)), `deformField`, `lineOffset`, `size`. Full style + point modes: [layers-styling.md](layers-styling.md). A DataAsset `time` config drives appear-by-year / trip comet-trail.

## assets{} — DataAssets (the geometry source)
A DataAsset holds inline GeoJSON **and/or** a directive that the resolver bakes into concrete geometry:
```json
"plot": { "id": "plot", "type": "geojson", "resolve": "building",
  "data": { "type": "FeatureCollection", "features": [ { "type": "Feature", "properties": {}, "geometry": { "type": "Polygon", "coordinates": [[ "..." ]] } } ] } }
```
Directives (`place`, `snap`, `resolve:"building"`, `query.kind:"road"|"boundary"|"poi"`, `url`, `icon:"search:…"`) are resolved into the file by `editgeo resolve` **before** preview/render. The directive catalogue + VN geography live in the **editgeo-data** skill. After resolve, `data` holds only concrete geometry.

## drawingOptions{} / markerTemplates{}
- **drawingOptions** — per-layer style: `fill{fillColor,opacity,patternType}`, `stroke{strokeColor,strokeWidth,dashVal,dashSpeed,strokeGapColor,strokeOutlineColor}`, `extrusion`, `pointMode`+`circle`/`heatmap`, and raw `paint`/`layout` passthrough. Full vocabulary: **[layers-styling.md](layers-styling.md)**.
- **markerTemplates** — per-label style: `font`, `fontSize`, `textColor{r,g,b,a}`, `backgroundColor{r,g,b,a}`. Referenced by `value.markerTemplateId`. See [markers.md](markers.md).

## Shot intents → IR (quick map)
| Intent | How |
|---|---|
| Fly to a place | `viewport` items with `flyTo` value; `place` or `lng/lat` |
| Orbit a building | `viewport` bearing hops + `buildings3d`; see [camera.md](camera.md) |
| Cinematic earth-dive | `viewport` from globe zoom ~2.5 → target; see [camera.md](camera.md) |
| Draw a route | `geojson` LineString + `snap`; `trimLineEnd`; see [recipes.md](recipes.md) |
| Highlight a plot/building | `geojson` Polygon + `DataAsset.resolve:"building"` |
| Highlight a ward / named road | `DataAsset.query.kind:"boundary"|"road"` (editgeo-data) |
| Subdivide land into lots (phân lô) | one `geojson` slot per lot, staggered `time`; numbers via a `marker` slot ([recipes.md](recipes.md)) |
| 3D buildings rise | `drawingOption.extrusion` on a footprint Polygon ([layers-styling.md](layers-styling.md)) |
| Clip imagery to a parcel / spotlight | `media` slot + `clipSlotId`/`clipMode` ([media.md](media.md)) |
| Data by year / moving trip | `DataAsset.time` + keyframed `now` ([temporal.md](temporal.md)) |
| Charts / photo pinned to a place | `marker` `chart`/`image`/`video` ([markers.md](markers.md)) |
| Markers / info boxes | `marker` items; see [markers.md](markers.md) |
| Fit a feature in frame | `viewport.value.fitFeature` |
| Measurements as text | compute → `marker` label; see [measurements.md](measurements.md) |
