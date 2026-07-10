# listenhear

A **single self-contained HTML file** OSINT dashboard: one map with every
public signal that's keyless and liftable, plus a scanner-style radio panel
with **three audio domains you can click and hear** — police/fire,
shortwave/ham, and aviation ATC.

No backend. No build step. No API keys. No `npm install`. Open `index.html`
directly in a browser.

## Run it

```
open index.html      # or just double-click it
```

Everything loads from CDNs (MapLibre GL + satellite.js) and fetches public
feeds client-side. Nothing is installed or served.

## What's on the map

Toggleable layers, refreshed on a timer, plotted from **keyless** public feeds:

| Group | Layers |
|-------|--------|
| Movement | USGS earthquakes · adsb.lol military aircraft · CelesTrak satellites (SGP4 propagated in-browser) · Amtrak trains |
| Hazard / env | NWS severe-weather alert polygons · Holocene volcanoes (curated static) |
| Geopolitics | DeepStateMap Ukraine frontline · abuse.ch malware C2 count |
| Badges | NOAA SWPC planetary **Kp** index · CISA **KEV** known-exploited CVE count |
| Reference | Right-click any point → dossier (Nominatim reverse-geocode + RestCountries + Wikidata head-of-state + Wikipedia summary) |
| Search | Place name or `lat,lon` via OSM Nominatim |

## The radio panel — the point of this tool

Three audio domains in one place (the gap no single existing tool fills):

1. **Police / Fire — OpenMHz.** Pick a trunked-radio system → live calls
   auto-play newest-last through one `<audio>` element. "Scan" cycles across
   systems. Unencrypted talkgroups only; expect the trunk-recorder delay.
2. **Shortwave / Ham — KiwiSDR.** Curated public HF receivers as map pins;
   click to open that node's native web tuner (`:8073`) in a new tab. (KiwiSDR
   audio is a WebSocket protocol with no plain URL, so this is the spec's
   deliberate "click-to-listen without a WS rabbit hole" option.)
3. **Aviation ATC — LiveATC.** Airport feeds; each row tries an inline mp3
   stream first (`<audio>` playback isn't CORS-gated) and falls back to
   LiveATC's official player if they reject the hotlink by Referer.

## Design notes & honest caveats

- **`<audio>` playback is not gated by CORS** — only JSON `fetch()` is. So the
  OpenMHz `.m4a` calls and LiveATC streams play cross-origin fine; only the
  OpenMHz *JSON API* carries a CORS/Cloudflare risk.
- **OpenMHz JSON API sits behind a Cloudflare challenge.** It serves fine from
  a normal residential browser but may be challenged from datacenter/VPN IPs —
  if the systems list stays empty, that's why. Audio is unaffected.
- **Feeds with no CORS header** (abuse.ch, CISA KEV) are kept but fail
  gracefully with a visible marker rather than being silently dropped.
- **Dropped because they require a key** (violating the keyless rule):
  OpenAQ v3, NASA FIRMS (MAP_KEY), APRS.fi, and all of OpenSky / aisstream /
  Shodan / Global Fishing Watch / Sentinel Hub.
- **Dropped because there's no keyless in-browser path:** raw APRS-IS (TCP) and
  Meshtastic MQTT (broker creds + protobuf).
- Every layer loader is defensive: one dead feed never breaks the board.

## Provenance

The list of public endpoints (and the keyless/keyed split) is lifted from
ShadowBroker's data-source table
([github.com/BigBodyCobain/Shadowbroker](https://github.com/BigBodyCobain/Shadowbroker),
AGPL-3.0). Only the *list of public endpoints* — public knowledge — is reused
here, not any of their code. ShadowBroker is the full-fidelity Docker build;
this is deliberately the opposite: minimal, keyless, one file, three audio
domains.
