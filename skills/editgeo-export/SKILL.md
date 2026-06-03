---
name: editgeo-export
description: Embed a editgeo map into a HyperFrames video and export the final file. Covers the two-phase model (live map preview while iterating, baked map.mp4 + GSAP overlays at export), why the map must be a baked <video> at render time, the composition contract (`<video data-ir=… src=…>`, mapMode variable, map.mp4 at root), screen-space overlays (title/price/brand/captions), and the one-command `editgeo export`. Use when wiring the map into a HyperFrames composition, adding overlays or captions over the map, or exporting the final video. For authoring the map IR see the editgeo skill; for the export command/flags see the editgeo-cli skill.
---

# editgeo → HyperFrames export

The map layer renders by the editgeo engine; titles, prices, captions and the brand render by HyperFrames (HTML + GSAP). Export stitches them. The map is an **internal artifact** — the user only ever sees the finished video.

## Two phases (never bake an MP4 per prompt)
1. **Iterate (instant):** the composition mounts the **live map** (engine `seek(t)` bound to the timeline) so the user tweaks and sees changes immediately — no MP4 bake. Drive iteration with `editgeo preview` or the composition in live mode.
2. **Export (once, when approved):** `editgeo export <dir>` bakes `map.mp4` from the IR, the composition **swaps the live map for `<video src=map.mp4>`**, and HyperFrames renders overlays + the video into the final file.

Why two phases: HyperFrames **cannot render a live WebGL map headless** (it breaks at render). So the map is baked to a video first, then HyperFrames composites over it. Because both modes run the *same* IR through the *same* engine, **the live preview equals the baked map** (WYSIWYG).

## The composition contract
The composition (`index.html`) declares the map layer with a `<video>` that carries BOTH the baked source and the IR it came from:
```html index.html
<video id="hfmap" class="clip" src="assets/maps/map.mp4" data-ir="assets/maps/map.json"
       data-start="0" data-duration="16.5" data-track-index="0" muted playsinline></video>
```
- `data-ir` — the source IR. `editgeo export` reads it, bakes `map.mp4` (caching by mtime), then renders.
- `data-duration` MUST equal the IR `output.duration` (= the baked map.mp4 length).
- The map `<video>` MUST live at the **root** composition — HyperFrames does not extract videos nested in sub-compositions.
- A `mapMode` variable selects live-vs-video: at render, `editgeo export` passes `--variables {"mapMode":"video"}` so the composition shows `<video>` and does not mount the live map.

Start from the template at `../editgeo/assets/composition.html` (copy it to the project as `index.html`, point the `<video>` at your IR, set `data-duration`, fill the overlay copy).

## Overlays (screen-space — NOT the map)
Title, price, brand, progress bar, captions are HyperFrames overlays in screen space (not geo-anchored markers). On-map labels/info-boxes belong in the IR instead (editgeo skill → markers).

**Listing copy = HyperFrames variables, not hard-coded text.** Declare them on the `<html>` root and read them in the script — don't find-and-replace strings:
```html index.html
<html data-composition-variables='[
  {"id":"brand","type":"text","label":"Brand (empty = none)","default":""},
  {"id":"title","type":"text","label":"Title","default":"4-BR Villa"},
  {"id":"price","type":"text","label":"Price","default":""}
]'>
```
```js index.html
const V = window.__hyperframes.getVariables();   // read ONCE at top
document.getElementById("brand").textContent = (V.brand ?? "").trim();
```
Set per render with `--variables` (the SAME object that carries `mapMode`):
```bash
hyperframes render . --variables '{"mapMode":"video","brand":"Nhà Phố Group","title":"Biệt thự 4PN","price":"12 tỷ"}'
```
**White-label:** `brand` defaults to `""` — empty shows NO brand, and an empty brand should not animate in. The brand is the **user's own** (their agency/agent) — ASK the user and pass it in `--variables`. **Never** put "EditBĐS" on a user's video; EditBĐS is the platform, not their brand. They live in the composition with GSAP. On-map labels/info-boxes belong in the IR instead (see the editgeo skill → markers). Narration + word-timed captions: `npx hyperframes tts script.txt -o assets/audio/vo.wav` then `npx hyperframes transcribe vo.wav` → a transcript the composition reads to highlight words.

## Composition authoring rules
Follow [references/composition-rules.md](references/composition-rules.md) (data-start / class="clip" / `window.__timelines` / determinism). Getting these wrong makes HyperFrames skip or freeze elements.

## Export
`editgeo export <compositionDir>` is the whole thing: resolve + bake `map.mp4` (cached if the IR is unchanged) → `hyperframes render` → final MP4. Pass listing copy with `--variables` — it's merged with `mapMode` (which is always forced to `"video"`):
```bash
editgeo export ./my-video --variables '{"brand":"Nhà Phố Group","title":"Biệt thự 4PN","price":"12 tỷ"}'
```
See the editgeo-cli skill for the other flags.
