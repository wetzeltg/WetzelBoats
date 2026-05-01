# WetzelBoats

A live maritime and wildlife dashboard for the southern Puget Sound, focused on the Port of Tacoma area. Built as a single-file web app for the Wetzel family's Google Pixel tablet in Washington state.

**Live URL:** `https://wetzeltg.github.io/WetzelBoats/`

---

## What It Shows

### Vessel Traffic (left panel)
- Live AIS (Automatic Identification System) feed of all commercial vessels in southern Puget Sound
- Vessels categorized as **Arriving**, **Departing**, **In Port** (docked at Port of Tacoma terminals), or **Anchored/Waiting** (at anchor in the water awaiting a berth)
- Ship cards show vessel name, photo (from MarineTraffic), type, flag, origin/destination, and speed
- All vessels sorted by distance from home (4910 La Hal Da Ave NE, Tacoma WA 98422)
- Stats strip always visible at top: Arriving / Departing / In Port / In Sound / Anchored counts
- Collapsible debug strip showing raw AIS data for troubleshooting

### Interactive Map (center)
- OpenStreetMap base layer centered on southern Puget Sound (zoom 11, centered at 47.3814, -122.4925)
- Home location marker (house icon) visible in lower portion of default view
- Custom top-down vessel icons that **rotate to face the ship's actual AIS heading**
- Custom whale/orca silhouette icons placed at reported sighting locations
- Click any map icon to open a popup with details
- Tap or hover a map icon → highlights the corresponding card in the side panel, and vice versa

### Right Panel
Three sections stacked vertically, all visible without scrolling:

**Weather · Tacoma** — compact single-line display: temperature, conditions, wind, humidity from Open-Meteo (no API key required, updates every 10 minutes)

**Whale & Orca Sightings** — live reports from The Whale Museum Hotline. Shows species, location, time, pod identity (for orcas), and individual count. When there are no sightings in the last 24 hours, displays a **Dad Joke of the Day** instead ("No whales today — but here's a whale of a joke!") fetched from icanhazdadjoke.com.

**Daddy Countdown** — family feature showing Todd's travel schedule between Florida and Washington. Displays a rotating photo of Todd (14 photos, cycles every 30 seconds) and one of three states:
- *Daddy is coming!* — countdown to next arrival at SEA, with flight number and date/time on same line
- *Daddy is HOME!* — countdown to departure, with flight number
- *Miss you Daddy* — between trips

### Namibia Fun Fact Banner
A full-width banner that slides up from the bottom of the screen, honoring the family's Namibian exchange student. Appears 10 seconds after page load, then every 2 minutes. Auto-dismisses after 15 seconds or can be closed with the ✕ button. Features:
- A mini SVG rendering of the Namibian flag (all 5 colors, 12-rayed golden sun)
- 20 curated uplifting, factual facts about Namibia covering wildlife, conservation, geography, and culture
- Deep blue background (`#003580`) with gold label text (`#FFCE00`) matching the flag colors

---

## Map Icons

### Vessels (top-down view, rotates to live AIS heading)
| Status | Color | Details |
|--------|-------|---------|
| Arriving | Blue | Pointed bow, parallel sides, flat stern + 4 wake lines trailing behind |
| Departing | Green | Same hull + 4 wake lines |
| In port | Amber | Same hull + anchor symbol (no wake — stationary) |
| Anchored/waiting | Purple | Same hull + faint drift wake + anchor symbol |

The anchor symbol: ring at top → vertical shank → horizontal stock crossbar → curved arms with arc midpoint connecting to shank base.

### Marine Mammals (top-down silhouettes, all same base outline)
| Species | Color | Distinguishing Features |
|---------|-------|------------------------|
| **Orca** | Black | White belly patch, white eye patches, white saddle patch, tall straight dorsal fin |
| **Gray whale** | Blue-gray | Heavy mottling/barnacle blotches, no dorsal fin, knuckle bumps along dorsal ridge |
| **Humpback** | Dark green | Lighter belly, very long swept-back pectoral fins, small hooked dorsal fin set far back, knobby tubercles on rostrum |

---

## Travel Schedule (Daddy Countdown)

Current hardcoded trips (update in `index.html` under `const TRIPS`):

| Flight | Direction | Date & Time | Notes |
|--------|-----------|-------------|-------|
| AS 1525 | FL → SEA | May 1, 2026 · 9:30 PM PDT | Arrives Seattle |
| AS 441 | SEA → FL | May 9, 2026 · 9:30 AM PDT | Departs Seattle |
| AS 427 | FL → SEA | May 29, 2026 · 10:28 PM PDT | Arrives Seattle |
| AS 394 | SEA → FL | May 31, 2026 · 9:18 PM PDT | Departs Seattle |

Photos are stored in the repo as `d1.jpg` through `d14.jpg` and rotate every 30 seconds. The photo display uses a blurred backdrop — the full photo is shown uncropped with a blurred version filling the background behind it (same technique as Instagram/Spotify).

To add more trips, edit the `TRIPS` array in `index.html`. To add more photos, upload files to the repo and update the `Array.from({length:14}...)` count.

---

## Auto-Refresh on Deployment

The app automatically detects when a new version has been deployed to GitHub Pages and reloads itself — no manual refresh needed on the tablet.

Every 2 minutes, the app fetches the live `index.html` from GitHub with cache-busting and compares the version number to the currently running version (stored in `CURRENT_VERSION`). If they differ, the app waits 2 seconds and performs a hard reload. Network errors are silently ignored.

**Important:** Every time the code is updated, `CURRENT_VERSION` must be bumped (e.g. `v4.7` → `v4.8`) for the detection to trigger correctly.

---

## Data Sources

| Data | Source | API Key Required |
|------|--------|-----------------|
| Live vessel positions & data | [AISStream.io](https://aisstream.io) WebSocket | Yes (free, hardcoded) |
| Weather | [Open-Meteo](https://open-meteo.com) | No |
| Whale sightings | [The Whale Museum Hotline](https://hotline.whalemuseum.org) | No |
| Ship photos | MarineTraffic (by MMSI, public thumbnails) | No |
| Dad jokes | [icanhazdadjoke.com](https://icanhazdadjoke.com) | No |
| Namibia facts | Hardcoded curated list (20 facts) | No |
| Map tiles | OpenStreetMap via Leaflet.js | No |

---

## API Key Setup

The AISStream API key is hardcoded in `index.html`:

```javascript
let aisKey = localStorage.getItem('aisKey') || 'YOUR_KEY_HERE';
```

The key is free — sign up at [aisstream.io](https://aisstream.io). Since the repo is public and the key is free with no billing, this is acceptable for a personal project. If the key needs to be regenerated, log into AISStream and create a new one.

The ⚙ **API Keys** button in the top-right header can also override the key via the browser's localStorage on any device.

---

## AIS Data Notes

The AIS stream sends two types of messages:

- **PositionReport** — arrives every few seconds per vessel: lat/lng, speed, heading, navigational status
- **ShipStaticData** — arrives every ~6 minutes per vessel: vessel name, type, destination, call sign, dimensions

Because static data arrives infrequently, ship cards may show `—` for Type, Flag, or Destination when first seen. These fields fill in automatically over 10–15 minutes as each vessel broadcasts its static update.

### Status Classification Logic
- **In port** — inside Port of Tacoma terminal boundaries (lat 47.24–47.30, lng -122.44 to -122.38) and speed < 1 knot, OR nav status = moored within port area
- **Anchored** — nav status = at anchor outside port, OR speed < 1 knot outside port boundaries
- **Arriving** — destination includes "TACOMA" or heading southbound (135°–210°)
- **Departing** — destination is a non-Tacoma port, or heading northbound

### AIS Bounding Box
Filtered to southern Puget Sound: `[[47.0, -122.8], [47.7, -122.2]]`

---

## Deployment

Single-file static web app — no server, no build step, no dependencies to install.

### GitHub Pages
1. Edit `index.html` in the GitHub web editor (click file → pencil icon → select all → paste → commit)
2. Bump `CURRENT_VERSION` in the JS to the next version number
3. GitHub Pages deploys within ~60 seconds
4. The tablet detects the new version within 2 minutes and reloads automatically

### Running Locally
Open `index.html` directly in Chrome. All APIs work from a local file context.

### Pixel Tablet (Washington)
The tablet has Chrome open to `https://wetzeltg.github.io/WetzelBoats/` installed as a home screen app (Chrome menu → Add to Home Screen). After any update deploys, the app auto-reloads within 2 minutes — no manual intervention needed.

---

## File Structure

```
index.html     — entire app (HTML + CSS + JS, ~100KB, self-contained)
README.md      — this file
d1.jpg         — Daddy photo 1
d2.jpg         — Daddy photo 2
...
d14.jpg        — Daddy photo 14
```

All SVG icon definitions (vessels and whales) are embedded directly in the JavaScript as template literals inside `index.html`.

---

## Features On The Roadmap

- **"Big Boat! Big Boat!"** — detect large container ships underway and trigger a sound + banner alert
- **Morning wakeup greetings** — timed messages for each girl when they typically arrive in the kitchen for breakfast
- **Flight status on travel days** — live updates on whether AS flights are on time, have departed, or have landed (requires RapidAPI / AeroDataBox key)
- **Cargo inference** — show "likely cargo" on ship cards based on vessel type and route

---

## Version History

| Version | Changes |
|---------|---------|
| v4.7 | Auto-refresh on deployment: app detects new version from GitHub every 2 minutes and reloads automatically |
| v4.6 | Namibia Fun Fact banner: slides up from bottom every 2 minutes, SVG Namibian flag, 20 curated facts, auto-dismisses after 15 seconds |
| v4.5 | All flight times updated with exact schedules (AS441, AS427, AS394) |
| v4.4 | Right panel layout overhauled: compact weather, whale section capped with scroll, daddy always visible; photo URL case fixed to WetzelBoats |
| v4.3 | Blurred backdrop photo display (full photo shown uncropped) |
| v4.2 | Daddy countdown moved to right panel below whale sightings; photo loading fixed |
| v4.1 | Flight number inlined with arrival date/time on same row |
| v4.0 | Exact WetzelBucks branding: white header, indigo `#4f46e5` accents, `#f7f8fa` background, system fonts |
| v3.9 | Dad Joke of the Day when no whale sightings |
| v3.8 | WetzelBoats rebrand (indigo theme, renamed from Puget Sound Dashboard) |
| v3.7 | Rotating daddy photos (14 photos, 30s interval) |
| v3.6 | Fixed JS crash (unescaped apostrophe); AeroDataBox flight lookup scaffolded |
| v3.5 | Daddy Countdown widget: photo, countdown, 3 states, Alaska Airlines flights |
| v3.0–3.4 | Wake line width, color, opacity, and length iterations |
| v2.9 | All 4 boat icons unified viewBox; dark wake lines |
| v2.5–2.8 | Wake line visibility fixes (viewBox clipping, stroke width) |
| v2.4 | Custom top-down SVG vessel icons rotate to AIS heading; whale silhouette icons |
| v2.3 | Single distance-sorted vessel list; scroll-to-card on hover |
| v2.2 | Photo flash fix; sticky stats bar |
| v2.1 | Map zoom/center set from live browser (zoom 11, 47.3814, -122.4925) |
| v2.0 | Toggleable debug panel |
| v1.9 | Fixed JS crash (duplicate `const anchored`) |
| v1.8 | Tighter bounding box; anchored/waiting category; 5-stat strip |
| v1.5–1.7 | AIS Blob parsing fix; vessel enrichment; flag table |
| v1.4 | Version number in header |
| v1.0–1.3 | Initial build through light theme |

---

## Known Limitations

- **Cargo data** — AIS does not transmit cargo contents; ship type gives a general indicator
- **Vessel enrichment** — VesselFinder/MyShipTracking lookups via CORS proxy are best-effort
- **Static data lag** — Type, Flag, Destination fields take up to 15 minutes to populate for new vessels
- **Whale API** — The Whale Museum Hotline occasionally goes offline; app shows Dad Joke fallback
- **Flight times** — Currently hardcoded; AeroDataBox integration scaffolded but requires a RapidAPI key
- **Touch vs hover** — Hover highlights work with a mouse; on the Pixel tablet, tap achieves the same result
- **Auto-refresh** — Requires `CURRENT_VERSION` to be bumped with each deployment to trigger correctly
