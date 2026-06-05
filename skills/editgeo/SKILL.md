---
name: editgeo
description: Author animated MAP videos for the EditBĐS / editgeo engine from natural language — fly-to, orbit, cinematic earth-dive, draw routes, highlight a building/boundary, subdivide land into lots (phân lô), grow 3D buildings, clip satellite imagery to a parcel, animate data over time (events by year / a moving trip), heatmaps, on-map charts (giá/m², unit mix), and labels/info boxes. Use when asked to "add a map", "make a map video", "fly to <place>", "highlight <road/building/ward>", "phân lô / chia nền", "show the route/travel time to ...", "show <data> by year / over time", "biểu đồ giá trên bản đồ", "clip ảnh theo ranh", "intro/showcase video for <project/area/line>", or any geographic shot for a real-estate (BĐS) or location video. You write ONE IR JSON; the map is DATA, never hand-written code. For the resolver/data directives (geocode, route snap, building footprint, road/boundary, POI, VN admin geography) see the editgeo-data skill; for CLI commands (login, resolve, preview, render, export) see the editgeo-cli skill; for embedding the baked map into a HyperFrames video with overlays/captions see the editgeo-export skill.
---

# editgeo — map videos as data

The map layer is **one IR JSON file**, validated by `schema/map-ir.schema.json`. You author and edit that file; tools resolve it, render it to MP4, and (optionally) embed it in a HyperFrames composition. **The map is DATA, never code** — never hand-write map-rendering JavaScript. If a need can't be expressed in the IR, the schema + engine get extended once in the engine package, never per-video.

## Approach

### Discovery (open-ended requests only)
For vague asks ("make a video for this listing", "intro for the metro line") settle intent before authoring:
- **Subject** — a property, a route, an area/ward, a transit line?
- **Platform & length** — TikTok/Reel vertical 15–30s? Landscape hero? Pace: *nhanh, dứt khoát* (fast, decisive) is the EditBĐS house feel.
- **Priority** — cinematic motion, data accuracy, or brand fidelity?

For specific asks ("highlight this plot", "fix the camera on scene 2"), skip discovery and edit the IR.

### Author the IR
Write against **`schema/map-ir.ts`** (typed) / **`schema/map-ir.schema.json`** (validated). Top-level shape:
`output` · `basemap` · `slots[]` · `assets{}` (DataAssets) · `drawingOptions` · `markerTemplates` · `images{}` · `media{}`.
Full field reference: **[references/schema.md](references/schema.md)** (it carries inline IR snippets for every shape).

**Before authoring anything cinematic, read [references/cinematography.md](references/cinematography.md)** — the EditBĐS house style (prescriptive defaults, easing-as-emotion, Build→Breathe→Resolve, the warm grade, and the anti-pattern list that separates a cinematic map from a generic "AI map"). It is the difference between *correct* and *good*.

Pick the shot(s) and follow the matching reference:
| Want | Reference |
|---|---|
| **House style: look, easing, pacing, grade, presets, anti-patterns** | **[references/cinematography.md](references/cinematography.md)** |
| Camera: fly-to / orbit / cinematic earth-dive / follow-route / fit-bounds | [references/camera.md](references/camera.md) |
| Layer style: fill/stroke, dash + marching-ants, pattern, road casing, 3D extrusion, circle/heatmap, raw paint passthrough | [references/layers-styling.md](references/layers-styling.md) |
| Markers, info boxes, icons, **rich cards (chart/image/video/html)** | [references/markers.md](references/markers.md) |
| Time-series: events by year, moving trip + comet trail (the `now` playhead) | [references/temporal.md](references/temporal.md) |
| Image/video painted on the ground; **clip imagery to a parcel/boundary** | [references/media.md](references/media.md) |
| Area/frontage/route-length/ETA/distance → printed labels | [references/measurements.md](references/measurements.md) |
| Recipes: journey, highlight a building/plot/division, draw a route, ward/road | [references/recipes.md](references/recipes.md) |
| Color grade, vignette, tilt-shift, terrain/fog/hillshade/light, basemap style | [references/fx-world.md](references/fx-world.md) |
| Real geometry (geocode, route, footprint, road/boundary, POI) + VN geography | **editgeo-data** skill |

### Workflow (two phases — see editgeo-cli + editgeo-export)
1. **Bake the data** — abstract refs (`place`, `snap`, `resolve:"building"`, `query`, `search:` icons) become concrete geometry: `editgeo resolve map.json`. *(editgeo-data)*
2. **Iterate on the live map** — `editgeo preview map.json` shows the real map; tweak the IR, refresh. Get the user's approval. **Do not bake an MP4 each prompt.** *(editgeo-cli)*
3. **Export once when approved** — `editgeo export <dir>` bakes the map MP4 and renders the final HyperFrames video (map + overlays/captions). *(editgeo-export)*

## Golden rules
1. **The map layer is the IR JSON.** One file. Everything (camera, layers, routes, highlights, markers, fx) lives there.
2. **Resolve before preview/render.** After `resolve`, the IR holds only concrete geometry — no unresolved directives. This is the Single Source of Truth: same IR → same pixels, offline, deterministic.
3. **Preview before export.** Show the live map and get approval first. Iterating is instant; exporting an MP4 is not — never bake per prompt.
4. **Deterministic.** No `Date.now`, no randomness, no in-render fetches. The baked IR renders the same every time.
5. **House feel:** fast, decisive, *cinematic* motion. **Always set `defaultEaser: "inOutQuart"` + `viewportSmoothing: 0.4`** (a camera that glides, not snaps). Warm satellite/dark look; labels at video scale (not web-UI opacity). Full gu: [references/cinematography.md](references/cinematography.md).

## Cinematic quality bar (the gu — enforce, don't skip)
A map that is *correct* but *generic* is the failure mode. These are hard rules — follow them unless the user asks otherwise. Full reasoning + presets: **[references/cinematography.md](references/cinematography.md)**.
- **Easing is the adverb — vary it.** ≥2 distinct easers across the timeline; the slowest move ~3× slower than the fastest. One easer on everything = "AI map".
- **Every shot: Build → Breathe → Resolve.** The camera never hard-stops — the breathe beat keeps a subtle drift (a few degrees of bearing / a hair of zoom) while markers/borders reveal.
- **Vary the camera move.** Not flyTo+zoom every time — orbit, pull-back, follow-route, fit-bounds, or a static hold with a marker reveal (pick by subject).
- **Never a naked frame.** Every IR has an `fx` slot (warm grade + vignette, subtle at the wide → lifted on arrival) and a `world` slot (golden light/fog). Every shot has ≥1 focal accent.
- **Markers at video scale, staggered, asymmetric** (entranceIn > exitOut; total stagger <500ms).
- **Don't:** start everything at `t=0` (offset the open 0.2–0.3s) · stack eased waypoints in an earth-dive (one segment, fixed center) · spin `bearing` 0→360 in one keyframe (hops <180°) · over-grade (sat/contrast ≤1.3).
- **Before `preview`, run the pre-preview checklist** at the end of cinematography.md.

## Code blocks are labeled with their file
When you show IR, label the block with the filename so it's clear what to write, e.g.:
```json map.json
{ "version": 1, "output": { "width": 1920, "height": 1080, "fps": 30, "duration": 8 }, "...": "..." }
```

## Requirements
Node 22+ / Bun, system Google Chrome, FFmpeg. MapTiler tiles + map-data resolvers route through the gated server when logged in (`editgeo login`); see editgeo-cli. The engine is the core; this skill is one way to drive it.
