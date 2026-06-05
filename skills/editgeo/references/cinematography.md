# Cinematography — the EditBĐS house style (read this BEFORE authoring)

This is not optional polish. A map that is *correct* (right place, right data) but *generic* looks like every other AI map — flat grade, one ease on every move, a flyTo+zoom on every shot, a hard stop, naked frames. That is the failure mode. This page is the **prescriptive gu**: use these defaults unless the user explicitly asks otherwise. The motion is the verb; the **easer is the adverb** — choose it deliberately.

**The look:** warm, cinematic, premium (flagship BĐS). Satellite or dark basemap, golden light, subtle-then-confident grade, smooth decisive camera. **The feel:** *nhanh, dứt khoát* — short holds, confident eases, never drifting aimlessly, never stuttering.

---

## 0. Two load-bearing knobs — set them on EVERY IR

These exist in the engine and fix "jerky/poor camera" — but only if you set them. Most ugly output omits both.

```json map.json
{ "version": 1, "defaultEaser": "inOutQuart", "viewportSmoothing": 0.4, "output": { "...": "..." } }
```
- **`viewportSmoothing`** (0–1, default 0) — extra easing layered on the camera path. **Set 0.35–0.5.** This is the difference between a camera that *glides* and one that snaps between keyframes. Without it the camera reads as robotic.
- **`defaultEaser`** — the fallback ease for any item without its own `easer`. **Set `"inOutQuart"`** (confident). Never leave it unset (engine falls back to flat `inOutQuad`).

---

## 1. Easing = emotion (vary it — same ease everywhere = "AI map")

The engine ships these (camera.md lists them). They are not interchangeable — each says something:

| Feel you want | Easer | Use for |
|---|---|---|
| Confident, decisive | `inOutQuart` · `inOutQuint` | the house default; fly-to, push-in, the arrival |
| Cinematic, weighty | `inOutCubic` · `inOutExpo` | the long earth-dive; dramatic reveals |
| Dreamy, atmospheric | `inOutSine` | slow holds, gentle pans, "drift with me" |
| Snappy pop (overshoot) | `inOutBack` | a marker/label punching in (small amplitude) |
| Constant rate | `linear` | orbit spin, marching dashes, a steady route draw |
| Playful (use rarely) | `inOutElastic` | one accent beat, never the camera |

**Hard rule (anti-monoculture):** a timeline that uses ONE easer on every move looks generated. Use **≥2 distinct easers**, and let the **slowest move be ~3× slower than the fastest** so the eye can tell what matters. Eases are like font weights — vary them on purpose.

---

## 2. Every shot: Build → Breathe → Resolve

A camera that flies in and **hard-stops** is the #1 reason maps feel cheap. Structure each shot in three beats:

- **Build (arrive)** — fly-to / dive / push-in with a confident ease (`inOutQuart`/`inOutCubic`). Decisive, 1.8–2.6s.
- **Breathe (hold + read)** — DON'T freeze. Hold 1.2–2.5s with **subtle continuous motion**: a gentle bearing drift (+5–8° over the hold, `linear`) or a tiny zoom push (+0.15–0.25). This is when markers/borders/labels reveal and the viewer registers the subject. *A scene with no breathe doesn't let the content land; a scene that's all build is a slideshow.*
- **Resolve** — settle on the final composition, or push decisively to the next subject. Exits faster than entrances.

```json map.json
{ "id": "cam", "type": "viewport", "items": [
  { "time": 0,   "easer": "inOutCubic", "value": { "lng": 106.70, "lat": 10.79, "zoom": 12, "pitch": 0,  "bearing": 0 } },
  { "time": 2.3, "easer": "inOutQuart", "value": { "lng": 106.7009, "lat": 10.7769, "zoom": 16.8, "pitch": 58, "bearing": -14 } },
  { "time": 5.0, "easer": "linear",     "value": { "lng": 106.7009, "lat": 10.7769, "zoom": 17.0, "pitch": 58, "bearing": -6 } }
] }
```
*(beat 1→2 = build; 2→3 = breathe: same subject, tiny zoom+bearing drift, never a freeze.)*

**Open offset:** don't start every layer at `t=0` — it reads as a jump cut. Hold the wide for one beat; begin the first move at `0.2–0.3s`.

---

## 3. Camera grammar — vary the move (NOT flyTo+zoom every time)

The single biggest "AI map" tell: every shot is a fly-to that zooms in. Pick the move that fits the subject:

| Subject | Move | Key params |
|---|---|---|
| A building / villa (hero) | **push-in + slow orbit** | `pitch` 56–62, `zoom` 16.5–17.5, `buildings3d:true`, bearing hops <180° |
| An area / ward / district | **pull-back or lateral track** + `fitFeature` | `pitch` 35–45, `zoom` 12.5–14, frame the boundary |
| A route / journey | **follow the line** (`followFeature` progress 0→1) or `fitFeature` | `pitch` 55–65, `rotateAlong:true` |
| The whole region (intro) | **cinematic earth-dive** | `projection:"globe"`, ONE eased segment, fixed center |
| Phân lô / a grid | **top-down-ish reveal** | `pitch` 25–40, zoom close so the grid fills frame |
| A single fact/landmark | **static hold + marker reveal** | camera still; the motion is the label punching in |

Rules that prevent stutter (also in camera.md): **earth-dive = one eased segment, center FIXED** (multiple eased pan+zoom waypoints stutter at each stop); **orbit = hops <180°** with `linear` and a fixed center (a single 0→360 spins 0° — shortest arc).

---

## 4. The default grade & light (warm cinematic)

A naked basemap is a flat frame. Every IR gets an `fx` slot + a `world` slot. Keep it **subtle at the wide shot, intensify on arrival** ("the camera found its subject").

```json map.json
"slots": [
  { "id": "grade", "type": "fx", "items": [
    { "time": 0,   "easer": "inOutSine", "value": { "saturation": 1.02, "contrast": 1.03, "brightness": 1.0,
        "vignette": { "strength": 0.18, "size": 0.95, "softness": 0.6 } } },
    { "time": 2.4, "easer": "inOutQuad", "value": { "saturation": 1.16, "contrast": 1.12, "brightness": 1.03, "sepia": 0.05,
        "vignette": { "strength": 0.4, "size": 0.82, "softness": 0.55 },
        "tiltShift": { "blur": 4, "blurMargin": 0.34, "focusArea": 0.3 } } } ] },
  { "id": "world", "type": "world", "items": [
    { "time": 0, "value": { "fogColor": { "r": 236, "g": 226, "b": 208, "a": 1 }, "fogDensity": 0.14,
        "lightColor": { "r": 255, "g": 242, "b": 222, "a": 1 }, "lightIntensity": 0.6,
        "lightPositionR": 1.2, "lightPositionA": 135, "lightPositionP": 34 } } ] }
]
```
- **Grade:** warm via small `sepia` (0.04–0.06) + a touch of `saturation`/`contrast`. **Never** push `saturation`/`contrast` past ~1.3 or it looks cheap-HDR. `tiltShift` only on close 3D dives.
- **Vignette:** always on (`strength` 0.15 wide → 0.4 on arrival) — pulls the eye to center.
- **Light:** golden-hour — warm `lightColor`, azimuth (`lightPositionA`) 120–150° for long building shadows. Pitch the camera so 3D buildings catch the light.
- **Basemap:** `satellite` for premium/aerial, `dark` for night/data, `white` only when labels must dominate.

---

## 5. Markers & labels — at video scale, staggered, asymmetric

- **Scale for video, not web.** Labels readable at 1080p+; use `markerTemplates` with real `fontSize` (≥18) and near-opaque `backgroundColor` (a ~0.9 alpha card, not a faint web overlay).
- **Asymmetry:** entrance longer than exit — `transitionIn` ~0.5s, `transitionOut` ~0.3s. Entrances build presence; exits just remove it.
- **Stagger by importance, total < 500ms.** Reveal the most important pin first, not DOM order. For phân-lô / POI: stagger `time` by `i*0.12–0.18` (the engine already staggers auto-POI markers).
- **Subtle life only:** a `sin` effector on `scale` (amp ≤ 0.06) or `opacity`. Keep amplitudes small — big wobble looks toy-like. `autoTransitionScale`/`autoTransitionOpacity` for the entrance.

---

## 6. Shot-style presets (start from one, then adapt)

Named house styles — each is a coherent camera + grade + pacing identity. Pick by brief:

- **Property Hero** — dive/push to the building, slow 120°-hop orbit during the breathe, warm grade + tiltShift, one bold price/title marker. `pitch` 58, `viewportSmoothing` 0.45, eases `inOutQuart`→`linear`.
- **Area Overview** — pull-back + `fitFeature` on the ward boundary, glowing border draw-on (`trimLineEnd`), 2–3 POI labels staggered. `pitch` 40, ease `inOutCubic`, lighter grade.
- **Route Journey** — `followFeature` along the snapped route, station info-boxes timed to camera arrival, comet-trail optional (temporal.md). `pitch` 60, `rotateAlong:true`, `linear` draw + `inOutQuart` arrivals.
- **Phân Lô Reveal** — close near-top-down, lots draw-on one-by-one (staggered slots), numbers as a marker slot, master-parcel spotlight (`media` `clipMode:"mask"`). ease `inOutQuart`, stagger 0.12.

---

## 7. Anti-patterns — the "AI map" tells (DON'T)

Each of these alone is the difference between cinematic and generated. Check your IR against all of them:

- ❌ **Every shot is flyTo + zoom-in.** Vary the move (§3) — orbit, pull-back, follow, static-reveal.
- ❌ **Camera hard-stops after arriving.** Always give the breathe beat a tiny drift; always set `viewportSmoothing`.
- ❌ **One easer on every move.** ≥2 distinct; slowest ≈3× the fastest (§1).
- ❌ **`defaultEaser` / `viewportSmoothing` unset.** Robotic camera.
- ❌ **Everything starts at `t=0`.** Offset the open 0.2–0.3s.
- ❌ **Earth-dive with several eased waypoints** → stutter. One segment, fixed center.
- ❌ **`bearing` 0→360 in one keyframe** → no spin. Hops <180°.
- ❌ **No `fx`/`world` slot** → flat, naked frame. Always grade + vignette + warm light.
- ❌ **Over-graded** (saturation/contrast >1.3) → cheap HDR. Subtle at wide, lift on arrival.
- ❌ **Web-opacity labels / tiny fonts.** Video scale, near-opaque cards.
- ❌ **Markers pop in/out simultaneously.** Stagger; entrance>exit.
- ❌ **Empty frame** — basemap + one fly, nothing else. Every shot needs ≥1 focal accent: a glowing border, a clipped parcel, a hero marker.

---

## 8. Pre-preview checklist (run this before `editgeo preview`)

- [ ] `defaultEaser: "inOutQuart"` and `viewportSmoothing: 0.35–0.5` set?
- [ ] ≥2 distinct easers across the timeline; slowest ~3× the fastest?
- [ ] Each shot has Build → Breathe → Resolve (a real hold with drift, no hard stop)?
- [ ] Open offset ≥0.2s (not everything at t=0)?
- [ ] Camera move varies (not flyTo+zoom every shot)?
- [ ] `fx` grade + vignette present; subtle wide → lifted on arrival; warm `world` light?
- [ ] Earth-dive = one segment fixed center? Orbit = hops <180°?
- [ ] Markers: entrance>exit, total stagger <500ms, video-scale fonts, subtle ambient?
- [ ] No empty frames — every shot has a focal accent?
- [ ] Nothing over-graded (sat/contrast ≤1.3)?
