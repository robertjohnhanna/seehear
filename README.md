# canifly

**Can I fly my drone here, now, and how high?**

One self-contained `index.html`. No backend, no build, no keys, no install —
open it in a browser. A dark map stays locked and centred on your location;
a panel carries a **flyability chart** and a distance-sorted **SITREP** of
everything around you. Works on iPhone, iPad, and desktop.

```
open index.html
```

## The chart

Altitudes 400 ft → surface (the Part 107 band) × NOW / +1h / +2h / +3h.
Green = fly, red = can't. The gate that bites prints its code in the cell:

| Code | Gate | Source |
|---|---|---|
| WIND / GUST | wind aloft / surface gust ≥ 27 mph | Open-Meteo |
| VIS | visibility < 3 SM | Open-Meteo |
| SKY | cloud base − 500 ft clearance caps the ceiling | Open-Meteo |
| KPx | Kp ≥ 7 grounds (GPS degraded) · Kp 5–6 ambers | NOAA SWPC |
| FAA | LAANC grid ceiling caps · defense TFR grounds | FAA ArcGIS |
| PROH / NSUF / PARK | prohibited / security zone / NPS land grounds | FAA · NPS |
| PCPN | radar echo within 5 mi grounds NOW · elsewhere on screen ambers | NEXRAD (pixel-sampled) |
| TRFC | a manned aircraft low in the 1 mi ring caps the column (stay 500 ft below it) | ADS-B |

**Fail-safe posture:** an unverifiable feed never silently reads "clear" — the
grid fails *open* but the NOW header, the rings, and a 📡 DATA card flag it
until the feed recovers. Feeds auto-retry; a failure never wipes the map.

## The rings

- **OPERATIONAL RANGE · 1 mi** (dashed) — the FAA query radius, the SITREP
  radius, and the red-flashing low-aircraft alert zone.
- **OPERATIONAL BUBBLE · 5 mi** (dotted) — traffic tracked, amber low-aircraft
  heads-up, and the precip grounding radius.

Both rings tint to the verdict — **green** clear, **amber** reduced or
unverified, **red** grounded. That colour *is* the go/no-go; the map carries no
other chrome. The view is fixed at a 10-mile framing, identical on every device.

## The SITREP

Info-only cards (no taps, no popups), tiered and distance-sorted: a red-flashing
**LOW AIRCRAFT** breach tops the pile, then red groundings, then the amber
low-aircraft heads-up, then everything else; below the weather card, whatever
else is **visible on screen but out of range** is listed as neutral context.

- Aircraft: ✈️/🚁 icons; tail tinted green civil / yellow military / red
  emergency. Low = under 900 ft AGL (400 ft band + 500 ft separation,
  QNH-corrected altitude minus terrain under the plane).
- Airspace: **restriction** (red — defense/prohibited/security) ≠
  **conditional** (amber — restricted/stadium, "no-fly if active") ≠
  **advisory** (MOA etc., info only) ≠ **authorization** (Class B/C/D/E,
  LAANC). Zones with floors above 400 ft are dropped.
- One weather card: promoted to a top hazard when a gate bites, otherwise the
  bottom conditions card. NWS warnings, SPC outlooks, Kp storms, solar flares.

On phones the map and panel are two full-screen pages — tap the map for the
panel, tap anywhere on the panel for the map, scroll to browse cards.

## Data

All keyless, client-side, refreshed on a 5 s pulse (each feed at its own
natural cadence). Aircraft: airplanes.live → adsb.fi. Weather: Open-Meteo.
Warnings: api.weather.gov. Radar: Iowa State Mesonet (drawn as raw pixels).
Airspace: FAA ArcGIS. Parks: NPS. Space weather: NOAA SWPC. Geo queries are
scoped to the visible map and refetch when your location moves.

## Location & privacy

GPS commits instantly when a ping is ≤ 65 m accurate, else the best ping
within 3 s; IP lookup is fallback-only. The map then follows you — jitter
holds, walking glides, a big correction re-frames. All camera input is dead;
data loads are positionally gated to your fix. Position never leaves the
browser except as anonymous lat/lon parameters to the public APIs.

## Caveats

- **Advisory, not authoritative.** Verify with official sources before flying.
- Hazard feeds (FAA, NWS, SPC, NEXRAD, NPS) cover the US; the rest is global.
- Imperial units throughout.
