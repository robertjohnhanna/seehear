# sitrep

**One map. Every keyless public signal, worldwide. The nearest emergency
broadcast, auto-tuned to where you are.**

A single self-contained HTML file OSINT dashboard: **worldwide** aircraft, live
earthquakes, military aircraft, emergency squawks, NWS + global disaster
warnings, the ISS, satellites, natural disasters, rocket launches, geopolitics
— flanked by two docked, full-height, equal-width side panels (**OBJECTS**
left, **SITREP** right). The SITREP panel also **auto-plays the closest
emergency broadcast** to your detected location and re-tunes it whenever that
location changes.

Every feed loads its **full** dataset (planes worldwide, the whole Starlink
constellation, all live seismicity) — never just a slice around the map centre.

No backend. No build step. No API keys. No `npm install`. **Open `index.html`
directly in a browser.** Fully responsive — works on iPhone, iPad, and desktop.

## Run it

```
open index.html      # or just double-click it
```

MapLibre GL and satellite.js load from CDN; every data feed is fetched
client-side from keyless public APIs. Nothing is installed or served.

## Layout — two docked side panels

The screen is anchored by two **unfloated, full-height, equal-width** panels
flush to the left and right edges, with the map centred in the open area
between them:

- **OBJECTS** (left) — the map-layer switchboard.
- **SITREP** (right) — the "at a glance" intelligence briefing, plus the
  auto-tuned emergency broadcast pinned at its foot.

There is no top chrome and no bottom bar. The map is otherwise chrome-free —
zoom by scroll/pinch, and a single **⌖ recenter button floats at the
bottom-right corner of the map**.

The whole interface is set in **upper case** (only measurement units stay in
natural case), at one uniform minimum text size, for a dense minimalist read.

## SITREP — situational awareness at a glance

On load, sitrep locates you (instant IP fix via ipwho.is, refined by GPS if
you allow the browser prompt — denial is silently tolerated), flies the map to
your position, drops a pulsing marker, and builds a live **SITREP briefing**
that refreshes every 60 s.

The SITREP reflects the **current map area** (viewport), not a fixed point —
pan or zoom and it re-reads what's in view (nearest aircraft, quakes,
disasters…).

- **Anomalies** — a correlation pass across every in-view feed surfaces the
  outliers at the top (aircraft squawking 7500/7600/7700, NWS warnings,
  red/orange GDACS alerts, M5+ quakes). When anything is flagged, the **SITREP
  title turns red** (warnings) or **amber** (alerts).
- ✈ every aircraft in view (auto-enabled worldwide layer)
- 🎖 military aircraft in view, with closest callsign
- 🚨 aircraft squawking 7700/7600/7500 worldwide, with distance to nearest
- 🌍 nearest earthquake and 🔥 nearest natural event, with distances
- 🌀 nearest global disaster (GDACS)

Every row has a VIEW action that flies the map there. The header carries a UTC
clock and your resolved location. Position is resolved client-side and never
leaves the browser.

Units are imperial (mi/ft, °F, mph) and the interface is set in a minimalist
all-caps style, with measurement units left in natural case.

## Emergency broadcast — NOAA Weather Radio, auto-tuned

Pinned at the foot of the SITREP panel is the nearest **NOAA Weather Radio
(NWR All Hazards)** station — the actual government emergency weather
broadcast, not a music or talk station. sitrep pulls the NWR feeds from
**radio-browser.info** (the keyless, CORS-friendly index), geolocates each one
(by its coordinates, or by the US state in its name when coordinates are
missing), plays the closest to your detected location through one `<audio>`
element, and **re-tunes automatically every time that location changes**. A
single **▶/⏸ button** stops and starts it; if a stream dies it drops that
station and falls back to the next-nearest.

> NWR is US-only, so outside the US the closest station is simply the nearest
> US transmitter. `<audio>` media loading is not gated by CORS, so the stream
> plays cross-origin without ACAO headers; browsers only defer autoplay until
> the first user gesture on the page.

## Live map layers

| Group | Layer | Source | Refresh |
|---|---|---|---|
| Air | Aircraft (**worldwide**, civil) | adsb global grid sweep | 90 s |
| | Military aircraft (worldwide) | adsb ADS-B `/mil` | 60 s |
| | Emergency squawks (7700/7600/7500) | adsb | 60 s |
| Surface | Amtrak trains | amtraker v3 | 60 s |
| Space | ISS live position | wheretheiss.at | 15 s |
| | Bright satellites | CelesTrak TLE → SGP4 propagated in-browser | 60 s |
| | GPS constellation | CelesTrak TLE → SGP4 | 2 min |
| | Starlink constellation (**full**) | CelesTrak TLE → SGP4 | 2 min |
| | Rocket launches (upcoming, at the pad) | Launch Library 2 | 60 min |
| Hazard | **Earthquakes (live)** — sized by strength, coloured by recency | EMSC seismicportal | 2 min |
| | **NWS warnings (US)** — warnings only, not watches | NOAA / api.weather.gov | 2 min |
| | Global disasters / warnings — quakes/cyclones/floods (alert-graded) | GDACS (UN/EC) | 30 min |
| | Natural events — wildfires, storms, volcanoes, ice | NASA EONET v3 | 10 min |
| | River / flood gauges (in view) | USGS water services | 10 min |
| | Holocene volcanoes | Smithsonian GVP (curated static) | — |

Earthquakes come from a **single** live source (EMSC) — the old USGS "24 h"
layer was a duplicate and was removed. **Warnings** are worldwide across the two
free, keyless, CORS-open warning APIs: **NWS** (every US warning type, watches
excluded) and **GDACS** (global multi-hazard alerts).

**Starts checked:** all aircraft (civil + military) — every other intel feed
loads in the background so the SITREP and anomaly detection stay complete
without cluttering the map.

> **Worldwide-aircraft caveat.** No keyless, CORS-open API serves every plane
> on Earth in one call (OpenSky's global feed is CORS-locked to its own domain
> and rate-limited; the CORS-open ADS-B mirrors cap point queries at 250 nm).
> sitrep therefore sweeps a global grid of busy-airspace anchors each cycle and
> merges the results — planes render across every continent, though the sparsest
> mid-ocean tracks can slip between anchors.

**Dossier:** right-click (desktop) or long-press (touch) anywhere →
reverse-geocoded place (Nominatim) + country profile (RestCountries) + head of
state (Wikidata SPARQL) + Wikipedia summary.

**Search:** place name or raw `lat,lon` via OSM Nominatim.

## Mobile

Designed for iPhone/iPad as first-class targets: bottom MAP / SITREP / OBJECTS
navigation with slide-up sheets, 44 px touch targets, safe-area (notch/home
indicator) insets, long-press dossier, `viewport-fit=cover`, momentum
scrolling. Desktop gets the two side-by-side docked panels.

## Design notes & honest caveats

- **`<audio>` playback is not gated by CORS** — only JSON `fetch()` is. So the
  emergency stream plays cross-origin fine; only the radio-browser *JSON API*
  carries any CORS risk, and it degrades silently with a retry.
- Every layer loader is defensive with silent auto-retry (exponential backoff):
  one dead feed never breaks the board, and failures show a neutral count
  rather than an error.
- **Dropped because they require a key** (violating the keyless rule):
  OpenAQ v3, NASA FIRMS (MAP_KEY), APRS.fi, OpenSky, aisstream, Shodan,
  Global Fishing Watch, Sentinel Hub.

## Provenance

The list of public endpoints (and the keyless/keyed split) draws on
ShadowBroker's data-source table
([github.com/BigBodyCobain/Shadowbroker](https://github.com/BigBodyCobain/Shadowbroker),
AGPL-3.0). Only the *list of public endpoints* — public knowledge — is reused,
not any of their code. ShadowBroker is the full-fidelity Docker build; sitrep
is deliberately the opposite: minimal, keyless, one file.
