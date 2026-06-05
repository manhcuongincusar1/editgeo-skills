# Media — images & video on the map, and clipping imagery to a shape

Paint a photo, render, site-plan, or video footage onto the ground (georeferenced by 4 corners), and optionally **clip it to a parcel/boundary polygon** — the "masked satellite imagery" look (image lives only inside the shape, with a glowing border).

## A `media` slot + `media{}` asset
```json
"media": { "plan": { "id": "plan", "kind": "image",
  "url": "https://…/site-plan.png",
  "coordinates": [[108.20,16.078],[108.262,16.078],[108.262,16.033],[108.20,16.033]] } },
"slots": [ { "id": "s-plan", "type": "media", "mediaId": "plan",
  "items": [ { "time": 0, "transitionIn": 1, "value": { "opacity": 0.9 } } ] } ]
```
- `kind`: `"image"` (single URL/data-URI) | `"video"` (one+ source URLs).
- `coordinates` — four `[lng,lat]` corners, clockwise from top-left (TL, TR, BR, BL). Warps with the camera in 3D.
- The slot's `items` keyframe `opacity` (fades in with `transitionIn`). Video is frame-synced to the timeline automatically.

## Clip imagery to a polygon (the Johnny-Harris look)
Add `clipSlotId` (id of a geojson slot whose first Polygon is the shape) + `clipMode`:
```json
"sat": { "id": "sat", "kind": "image", "url": "…", "coordinates": [ "…4 corners…" ],
  "clipSlotId": "s-parcel", "clipMode": "clip" }
```
- `clipMode: "clip"` — image shows **only inside** the shape; basemap kept outside. (Image should be ~axis-aligned; CORS-enabled — a tainted/cross-origin image falls back to unclipped.)
- `clipMode: "mask"` — **spotlight**: everything OUTSIDE the shape is dimmed by `maskColor` (`{r,g,b,a}`) at `maskOpacity` (default 0.82); basemap stays bright inside. Great for "highlight this plot" on a real satellite basemap.

Pair the clip shape's own `geojson` slot with a glowing border (stroke + `trimLineEnd` reveal — layers-styling.md) so the parcel reads as a framed hero. Z-order: media sits under vector overlays, so borders/markers draw on top.

## Determinism
The image/video loads once at init; render is then a pure function of time. For a fully offline SSoT render, inline images as data-URIs (or host on the proxied tile server). Never autoplay video — the engine seeks `currentTime` to the timeline.
