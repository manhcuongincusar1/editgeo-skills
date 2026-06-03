# HyperFrames composition rules

This page is the contract a composition must follow so HyperFrames renders it correctly. Break one of these and elements get skipped, frozen, or desynced from the map.

## Timing attributes
Every element that animates needs:
- `data-start` — when it appears (seconds, relative to its track).
- `data-duration` — how long it lives.
- `data-track-index` — its track (video on one, audio on another, etc.).

## class="clip"
Any element with timing **MUST** have `class="clip"` — the framework uses it to control visibility. Timed element without `clip` → it won't be shown/hidden correctly.

## Paused timeline on window.__timelines
The GSAP timeline must be **paused** and registered so the renderer can seek it:
```js index.html
window.__timelines = window.__timelines || {};
window.__timelines["main"] = gsap.timeline({ paused: true });   // id matches data-composition-id
```
The renderer seeks via `window.__timelines[id].totalTime(t)` — it does not play the timeline itself.

## Video + audio
- Map `<video>` uses `muted` + `playsinline`; put narration in a separate `<audio class="clip">` track.
- The map `<video>` must be at the **root** composition (sub-comp videos are not extracted). `data-duration` on it = IR `output.duration`.

## Determinism
Only deterministic logic — **no** `Date.now()`, **no** `Math.random()`, **no** render-time network fetches. Same composition + same inputs → same frames. (Fetching a local `transcript.json` at build time is fine; fetching remote data during render is not.)

## Map mode switch
Gate live-map vs baked-video on the `mapMode` variable so the same file previews live and renders from `<video>`:
```js index.html
const vars = (window.__hyperframes?.getVariables?.() || window.__hfVariables || {});
const RENDER = vars.mapMode === "video";
if (RENDER) { liveEl.style.display = "none"; }          // show <video src=map.mp4>
else { video.style.display = "none"; /* mount the live MapEngine, seek on tick */ }
```
`editgeo export` passes `--variables {"mapMode":"video"}` at render time.
