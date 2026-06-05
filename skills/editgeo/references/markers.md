# Markers — labels, info boxes, icons

This page teaches the `marker` slot: text anchored to coordinates, two-line info boxes, and icons. Markers are geo-anchored (they stick to lng/lat as the camera moves). Screen-anchored text (title/price/brand) is NOT a marker — it lives in the HyperFrames overlay (editgeo-export skill).

## A marker item
```json
{ "id": "m-villa", "type": "marker", "items": [
  { "time": 2.4, "duration": 6, "transitionIn": 0.5, "transitionOut": 0.4, "easer": "inOutBack",
    "value": { "markerTemplateId": "villa", "icon": "★", "text": "4-BR Villa · 320 m²",
               "lng": 106.7009, "lat": 10.7769, "autoTransitionOpacity": true, "autoTransitionScale": true },
    "animators": [ { "name": "breathe", "effectors": [ { "type": "sin", "propName": "scale", "amp": 0.05, "dur": 2.2 } ] } ] }
] }
```
`value` fields:
- `lng/lat` — anchor (or `place` for the resolver to geocode — see editgeo-data).
- `text` — line 1 (title). `subtitle` — line 2 (a short description / info box). Either or both.
- `icon` — see below. `markerTemplateId` — style from `markerTemplates`.
- `autoTransitionOpacity` / `autoTransitionScale` — fade/scale with the keyframe's `transitionIn`.
- `animators[]` — subtle life: `sin`/`blink`/`wiggle` effectors on `scale`/`opacity` (keep amplitudes small).

## Info box (2 lines)
Add `subtitle` for a station/place card — tên + mô tả ngắn:
```json
{ "value": { "text": "Ga Bến Thành", "subtitle": "Đầu tuyến · trung tâm Q.1", "lng": 106.698, "lat": 10.772 } }
```

## Rich content card (chart / image / html / video)
A marker card can hold rich content anchored to lng/lat (upright, camera-tracked):
```json
{ "value": { "lng": 106.70, "lat": 10.776, "text": "Giá/m² (triệu)",
  "chart": { "type": "bar", "title": "2020→2025", "color": { "r": 255,"g":205,"b":70,"a":1 },
             "data": [ {"value":28},{"value":41},{"value":50},{"value":62},{"value":78} ] } } }
```
- `chart` — inline-SVG chart (`type`: `"bar"`|`"line"`|`"donut"`; `data:[{label?,value,color?}]`; `title?`,`color?`,`max?`). Deterministic, no dependency — price trends, unit mix, growth.
- `image` — `<img>` URL/data-URI in the card (`imageWidth?` px). `video` — frame-synced `<video>` (no autoplay). `html` — raw HTML in the card body.
*(Geo-anchored. For a photo/video painted ON the ground by 4 corners, use the `media` slot — see media.md.)*

## Icons
`icon` accepts:
- An **emoji** — `"🚇"`, `"★"` (used as-is, no resolve).
- An **Iconify ref** — `"mdi:train"`, `"mdi:hospital-box"`.
- A **search** — `"search:metro station"` → the resolver finds the best icon.
Iconify/search icons are baked to inline `iconSvg` by `editgeo resolve`; emoji are left as-is. (Resolver details: editgeo-data.)

## markerTemplates
Define a reusable label style, reference it by `markerTemplateId`:
```json
"markerTemplates": {
  "villa": { "id": "villa", "font": "Inter", "fontSize": 20, "textColor": { "r": 30, "g": 25, "b": 0, "a": 1 }, "backgroundColor": { "r": 255, "g": 205, "b": 50, "a": 0.96 } },
  "poi":   { "id": "poi",   "font": "Inter", "fontSize": 15, "textColor": { "r": 240, "g": 245, "b": 255, "a": 1 }, "backgroundColor": { "r": 16, "g": 22, "b": 40, "a": 0.86 } }
}
```

## POI markers (auto from points)
A `geojson` slot whose DataAsset is a Point FeatureCollection (e.g. from `query.kind:"poi"`) auto-spawns one marker per point, staggered, using each feature's `name`/`icon`. You don't write a marker slot for those — the points become markers.

## Measurements as labels
Distances, areas, frontage, ETA are NOT a schema feature — you compute the number and drop a normal marker with the literal text. See [measurements.md](measurements.md).
