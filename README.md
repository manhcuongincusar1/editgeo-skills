# editgeo skills

Agent Skills that teach a coding agent (Claude Code, Cursor, Codex, Gemini CLI, Copilot, OpenCode, Zed…) to make **map videos for Vietnam real estate** — describe a place, the agent writes one IR JSON, and the [`editgeo`](https://www.npmjs.com/package/@editbds/cli) engine renders a map video locally. Built to drop into the [HyperFrames](https://github.com/heygen-com/hyperframes) video ecosystem.

> Map = DATA. You never hand-write map code — everything is one IR JSON.

## Install

**Any agent (Vercel `skills`):**
```bash
npx skills add manhcuongincusar1/editgeo-skills
```

**Claude Code (plugin marketplace):**
```
/plugin marketplace add manhcuongincusar1/editgeo-skills
/plugin install editgeo@editbds
```

Then install the engine CLI (the renderer — obfuscated, not in this repo):
```bash
npm i -g @editbds/cli
editgeo login
```

## Skills

| Skill | What it does |
|---|---|
| **editgeo** | Author the map IR from natural language (fly-to, orbit, earth-dive, routes, highlights, markers, labels). |
| **editgeo-cli** | The `editgeo` dev loop — login, resolve, preview, render, export. |
| **editgeo-data** | Resolver directives (geocode, route snap, building footprint, road/boundary, POI) + Vietnam geography (2025 ward reform). |
| **editgeo-export** | Embed the baked map into a HyperFrames composition with overlays/captions and export the final video. |

## How it works

```
describe a place → agent writes map.json (IR) →
  editgeo resolve   (bake geocode/route/footprint into the IR)
  editgeo preview   (live map you tweak)
  editgeo export    (bake map.mp4 → HyperFrames overlays → final video)
```

The map renders **locally** and deterministically. The rendering engine ships as the npm package `@editbds/cli`; this repo is just the skills (instructions) that drive it.

— [editbds.com](https://editbds.com)
