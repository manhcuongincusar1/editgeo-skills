# Measurements → labels

This page teaches how to put real numbers on the map — area, frontage × depth, route length, ETA, distance. These are **not** a schema feature: YOU compute the number from geometry you already hold, then emit a normal `marker` with the literal text. There is no `measure` directive. Compute, round, place.

## Formulas (compute in your head / a scratch calc, then write the label)
```js
const R = 6371000, rad = d => d * Math.PI / 180;
// distance between two [lng,lat], metres
const haversine = (a, b) => { const [x1,y1]=a,[x2,y2]=b, dφ=rad(y2-y1), dλ=rad(x2-x1),
  s=Math.sin(dφ/2)**2 + Math.cos(rad(y1))*Math.cos(rad(y2))*Math.sin(dλ/2)**2;
  return 2*R*Math.asin(Math.sqrt(s)); };
// line length, metres (LineString coords)
const lineLen = c => c.slice(1).reduce((m,p,i)=> m + haversine(c[i],p), 0);
// local metres-per-degree at a latitude (planar approx, fine at plot scale)
const mPerDeg = lat => ({ x: 111320*Math.cos(rad(lat)), y: 110540 });
// polygon ring [[lng,lat],…] (closed): area m² (shoelace) + bbox dims (frontage × depth) + centroid
function plot(ring) { const lat0=ring[0][1], {x:mx,y:my}=mPerDeg(lat0);
  const p=ring.map(([lng,lat])=>[lng*mx,lat*my]); let A=0;
  for (let i=0;i<p.length-1;i++) A += p[i][0]*p[i+1][1] - p[i+1][0]*p[i][1];
  const area=Math.abs(A)/2;
  const lngs=ring.map(c=>c[0]), lats=ring.map(c=>c[1]);
  const width=(Math.max(...lngs)-Math.min(...lngs))*mx;   // E–W frontage
  const depth=(Math.max(...lats)-Math.min(...lats))*my;   // N–S depth
  const cx=lngs.reduce((s,v)=>s+v,0)/lngs.length, cy=lats.reduce((s,v)=>s+v,0)/lats.length;
  return { area, width, depth, centroid: [cx, cy] }; }      // centroid = where to anchor the label
// midpoint of a line (label seat for a route), returns [lng,lat]
function lineMid(c) { const half=lineLen(c)/2; let d=0;
  for (let i=1;i<c.length;i++){ const seg=haversine(c[i-1],c[i]);
    if (d+seg>=half){ const t=(half-d)/seg; return [c[i-1][0]+(c[i][0]-c[i-1][0])*t, c[i-1][1]+(c[i][1]-c[i-1][1])*t]; } d+=seg; }
  return c[c.length-1]; }
const km = m => m>=1000 ? (m/1000).toFixed(1)+" km" : Math.round(m)+" m";  // format helper
```

## Route ETA
The route snap (`snap` directive, resolved by editgeo-data) returns OSRM `routes[0].duration` (s) and `distance` (m) — read them, don't recompute. Then label, e.g. `"→ Sân bay · 12 km · 18 phút"`.

## Place the label
Anchor at the geometry's natural seat — the **centroid** for a plot, the **lineMid** for a route/line:
```json
{ "type": "marker", "items": [
  { "time": 2, "value": { "text": "320 m² · 20×16 m", "lng": 106.7009, "lat": 10.7769 } }
] }
```
(lng/lat = `plot().centroid` for an area, `lineMid(coords)` for a route.)

Round to what a buyer reads — `320 m²`, `12 km · 18 phút`, `20×16 m` — not raw float precision.
