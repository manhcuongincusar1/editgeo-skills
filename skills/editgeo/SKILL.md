---
name: editgeo
description: Author animated MAP videos for the EditBĐS / editgeo engine from natural language — fly-to, orbit, cinematic earth-dive, draw routes, highlight a building or boundary, mark places, show paths and travel times, with on-map labels and info boxes. Use when asked to "add a map", "make a map video", "fly to <place>", "highlight <road/building/ward>", "show the route/paths to ...", "intro video for <area/line>", or any geographic shot for a real-estate (BĐS) or location video. You write ONE IR JSON; the map is DATA, never hand-written code. For the resolver/data directives (geocode, route snap, building footprint, road/boundary, POI, VN admin geography) see the editgeo-data skill; for CLI commands (login, resolve, preview, render, export) see the editgeo-cli skill; for embedding the baked map into a HyperFrames video with overlays/captions see the editgeo-export skill.
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
`output` · `basemap` · `slots[]` · `assets{}` (DataAssets) · `drawingOptions` · `markerTemplates`.
Full field reference: **[references/schema.md](references/schema.md)**. Example IRs: **[assets/examples/](assets/examples/)**.

Pick the shot(s) and follow the matching reference:
| Want | Reference |
|---|---|
| Camera: fly-to / orbit / cinematic earth-dive / follow-route / fit-bounds | [references/camera.md](references/camera.md) |
| Markers, on-map labels, 2-line info boxes, icons | [references/markers.md](references/markers.md) |
| Area/frontage/route-length/ETA/distance → printed labels | [references/measurements.md](references/measurements.md) |
| Recipes: journey, highlight a building/plot, draw a route, highlight a ward/road | [references/recipes.md](references/recipes.md) |
| Color grade, vignette, tilt-shift, terrain/fog/light, basemap style | [references/fx-world.md](references/fx-world.md) |
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
5. **House feel:** fast, decisive motion; white/clean or satellite basemap; labels at video scale (not web-UI opacity).

## Code blocks are labeled with their file
When you show IR, label the block with the filename so it's clear what to write, e.g.:
```json map.json
{ "version": 1, "output": { "width": 1920, "height": 1080, "fps": 30, "duration": 8 }, "...": "..." }
```

## Requirements
Node 22+ / Bun, system Google Chrome, FFmpeg. MapTiler tiles + map-data resolvers route through the gated server when logged in (`editgeo login`); see editgeo-cli. The engine is the core; this skill is one way to drive it.
