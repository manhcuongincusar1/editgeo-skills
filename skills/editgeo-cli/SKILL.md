---
name: editgeo-cli
description: The editgeo CLI dev loop — `editgeo` for account (login, whoami, buy), data baking (resolve), live iteration (preview), and output (render, export). Use when running any of these commands, wiring the login/billing flow, or troubleshooting the local render environment (Chrome, FFmpeg, MapTiler key). For authoring the IR JSON see the editgeo skill; for the data directives that `resolve` bakes see the editgeo-data skill; for the HyperFrames export details see the editgeo-export skill.
---

# editgeo CLI

`editgeo` is the single command surface. It runs everything locally on the user's machine and routes map-data calls through the gated EditBĐS server (metered against the user's plan). Install: `npm i -g @editbds/cli` (ships the engine bundled + obfuscated). In the monorepo, the same commands exist as `bun run <cmd>` from the repo root.

## Commands
```bash
# account
editgeo login            # browser → Google → mints an API key → ~/.editgeo/config.json
editgeo whoami           # show the logged-in account
editgeo logout           # remove the local key
editgeo buy              # open editbds.com/app to buy a Map plan / template

# make a video (map.json = the IR)
editgeo resolve <ir>     # bake directives (geocode/route/footprint/road/boundary/POI/icon) into the IR
editgeo preview <ir>     # live map in the browser — iterate, no MP4 bake
editgeo render  <ir>     # bake the deterministic map MP4
editgeo export  <dir>    # bake the map + render the final HyperFrames video (one command)
editgeo export  <dir> --variables '{"brand":"…","title":"…","price":"…"}'   # set listing copy (merged with mapMode)
```
`<ir>` is a path (`map.json`) or an example name. The IR may be the first positional arg or `--src <ir>`.

## Workflow (each step is a verb)
1. **Author** — write/edit the IR JSON (see the `editgeo` skill).
2. **Resolve** — `editgeo resolve map.json` → bakes real geometry (see `editgeo-data`). Re-run when you change a directive.
3. **Preview** — `editgeo preview map.json` → the live map. Iterate with the user. **Do not bake an MP4 each prompt.**
4. **Export** — when approved, `editgeo export <compositionDir>` → final video (see `editgeo-export`).

## Login & billing
- `resolve` (and any real map-data call) needs an **active Map plan**. If `editgeo whoami` shows no account, tell the user to run `editgeo login`; if the plan is expired, `editgeo buy`.
- `login` opens a loopback flow: the browser signs in with Google at `editbds.com/cli-login`, which posts the API key back to a localhost port. The key is stored at `~/.editgeo/config.json` (mode 600). The CLI injects it as `EDITGEO_API` / `EDITGEO_KEY` when running the engine, so map-data goes through the gated server.

## Environment / troubleshooting
- **Chrome** — render/preview drive system Google Chrome (or Chromium). Override with `CHROME_PATH` / `PUPPETEER_EXECUTABLE_PATH`.
- **FFmpeg** — required to mux frames into MP4. Install via the OS package manager.
- **MapTiler key** — basemap tiles. Read from `.env` (`MAPTILER_KEY`) or `--key`. A blank key shows a 403 on the basemap style — fix the key.
- **`--strict`** on render fails if the IR still has unresolved directives (run `resolve` first).
- **Flags** — `--out <file>`, `--ss <n>` (supersample), `--w`/`--h` (viewport), `--port`.

## Render is two phases, on purpose
Users never accept the first prompt's result, so the loop is built around **instant live preview** (no bake) and a **single export** at the end. Don't bake an MP4 to show progress — use `preview`. Details: the `editgeo-export` skill.
