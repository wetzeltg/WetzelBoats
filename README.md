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

**Whale & Orca Sightings** — live reports from the Acartia API (aggregates Orca Network data for the Salish Sea). Shows species, location, time, pod identity (for orcas), and individual count. Filtered to last 72 hours and Puget Sound/Salish Sea geographic bounds. Cards sorted by distance from home, then by recency on ties. Orca cards have a black left border; whale cards have a gray left border. When there are no sightings, displays a **Dad Joke of the Day** instead ("No whales today — but here's a whale of a joke!") fetched from icanhazdadjoke.com, plus a link to orcanetwork.org.

**Daddy Countdown** — family feature showing Todd's travel schedule between Florida and Washington. Displays a rotating photo of Todd (14 photos, cycles every 30 seconds) with blurred backdrop fill. One of three states:
- *Daddy is coming!* — countdown to next arrival at SEA, with flight number and date/time on same line
- *Daddy is HOME!* — countdown to departure, with flight number
- *Miss you Daddy* — between trips

### Namibia Fun Fact Banner
A full-width banner that slides up from the bottom of the screen, honoring the family's Namibian exchange student. Appears 10 seconds after page load, then every 2 minutes. Auto-dismisses after 15 seconds or can be closed with ✕. Features:
- A mini SVG rendering of the Namibian flag (all 5 colors, 12-rayed golden sun)
- 20 curated uplifting, factual facts about Namibia covering wildlife, conservation, geography, and culture
- Deep blue background (`#003580`) with gold label text (`#FFCE00`) matching the flag colors

---

## Map Icons

### Vessels (top-down view, rotates to live AIS heading)
| Status | Color | Details |
|--------|-------|---------|
| Arriving | Blue | Pointed bow, parallel sides, flat stern + 4 wake lines |
| Departing | Green | Same hull + 4 wake lines |
| In port | Amber | Same hull + anchor symbol (no wake) |
| Anchored/waiting | Purple | Same hull + faint drift wake + anchor symbol |

### Marine Mammals (top-down silhouettes, all same base outline)
| Species | Color | Distinguishing Features |
|---------|-------|------------------------|
| **Orca** | Black | White belly patch, white eye patches, white saddle patch, tall straight dorsal fin |
| **Gray whale** | Blue-gray | Heavy mottling/barnacle blotches, no dorsal fin, knuckle bumps along dorsal ridge |
| **Humpback** | Dark green | Lighter belly, very long swept-back pectoral fins, small hooked dorsal fin, knobby tubercles on rostrum |

---

## Travel Schedule (Daddy Countdown)

Current hardcoded trips (update in `index.html` under `const TRIPS`):

| Flight | Direction | Date & Time | Notes |
|--------|-----------|-------------|-------|
| AS 1525 | FL → SEA | May 1, 2026 · 9:30 PM PDT | Arrives Seattle |
| AS 441 | SEA → FL | May 9, 2026 · 9:30 AM PDT | Departs Seattle |
| AS 427 | FL → SEA | May 29, 2026 · 10:28 PM PDT | Arrives Seattle |
| AS 394 | SEA → FL | May 31, 2026 · 9:18 PM PDT | Departs Seattle |

Photos stored as `d1.jpg` through `d14.jpg`, rotate every 30 seconds with blurred backdrop fill technique.

To add trips: edit `const TRIPS` in `index.html`.
To add photos: upload to repo and update `Array.from({length:14}...)` count.

---

## Auto-Refresh on Deployment

Every 2 minutes the app fetches the live `index.html` from GitHub with cache-busting, compares the version number to `CURRENT_VERSION`, and hard-reloads if they differ. Network errors are silently ignored.

**Important:** Always bump `CURRENT_VERSION` (e.g. `v5.2` → `v5.3`) with each deployment for detection to trigger correctly.

---

## Data Sources

| Data | Source | API Key Required |
|------|--------|-----------------|
| Live vessel positions & data | [AISStream.io](https://aisstream.io) WebSocket | Yes (free, hardcoded) |
| Whale/orca sightings | [Acartia.io](https://acartia.io) — aggregates Orca Network data | Yes (free, hardcoded) |
| Weather | [Open-Meteo](https://open-meteo.com) | No |
| Ship photos | MarineTraffic (by MMSI, public thumbnails) | No |
| Dad jokes | [icanhazdadjoke.com](https://icanhazdadjoke.com) | No |
| Namibia facts | Hardcoded curated list (20 facts) | No |
| Map tiles | OpenStreetMap via Leaflet.js | No |

---

## API Keys in Code

```javascript
// AIS vessel tracking
let aisKey = localStorage.getItem('aisKey') || 'YOUR_AIS_KEY';

// Acartia whale sightings
const ACARTIA_KEY = 'YOUR_ACARTIA_KEY';

// Auto-refresh version detection
const CURRENT_VERSION = 'v5.3';

// GitHub Pages URL for version check
const GITHUB_URL = 'https://wetzeltg.github.io/WetzelBoats/index.html';
```

---

## AIS Data Notes

- **PositionReport** — every few seconds: lat/lng, speed, heading, nav status
- **ShipStaticData** — every ~6 minutes: name, type, destination, call sign
- Type/Flag/Destination fields take up to 15 minutes to populate for new vessels
- Bounding box: southern Puget Sound `[[47.0, -122.8], [47.7, -122.2]]`

### Status Classification
- **In port** — inside Port of Tacoma terminals (lat 47.24–47.30, lng -122.44 to -122.38), speed < 1 kn
- **Anchored** — nav status = at anchor, OR speed < 1 kn outside port
- **Arriving** — destination includes "TACOMA" or heading southbound (135°–210°)
- **Departing** — non-Tacoma destination or heading northbound

---

## Deployment

Single-file static web app — no server, no build step.

### GitHub Pages Workflow
1. Edit `index.html` (pencil icon in GitHub web editor → select all → paste → commit)
2. Bump `CURRENT_VERSION` in the JS
3. GitHub Pages deploys in ~60 seconds
4. Tablet detects new version within 2 minutes and reloads automatically

### Keeping Screen On (Pixel Tablet)
The app uses the **Wake Lock API** to request the screen stay on while the page is active. For guaranteed always-on behavior when plugged in:
- Settings → About tablet → tap Build number 7 times → Developer Options → **Stay awake while charging**

### Installing as PWA (Fullscreen, No Browser Chrome)
In Chrome on the Pixel tablet: three-dot menu → **Add to Home Screen**. Launches fullscreen from the home screen icon with no address bar or tabs.

---

## File Structure

```
index.html     — entire app (HTML + CSS + JS, ~100KB, self-contained)
README.md      — this file
d1.jpg         — Daddy photo 1
...
d14.jpg        — Daddy photo 14
```

All SVG icon definitions (vessels and whales) are embedded as JS template literals in `index.html`.

---

## Future Features Roadmap

### Mobile-Responsive Layout (designed, not yet built)
Full redesign for phones while keeping tablet layout unchanged:
- **Portrait mode** — map always visible, bottom sheet slides up when tabs tapped. Cards scroll horizontally, sorted by distance from current map center. Tapping a map icon opens the relevant tab and scrolls to that card.
- **Landscape mode** — map always visible, side drawer slides in from left. Cards scroll vertically.
- **Tabs:** 🚢 Ships · 🐋 Whales · 👨 Dad (+ Namibia facts folded in) · possibly 🌍 Notifications
- **Smart map linking** — map pan events re-sort cards live by distance from new center
- Namibia banner moves into Dad tab on mobile (conflicts with bottom sheet on small screens)

### "Big Boat! Big Boat!" Alert
Detect large container ships underway in the bounding box and trigger a sound + animated banner ("BIG BOAT! BIG BOAT! 🚢"). Trigger criteria: vessel type = Container, speed > 5 kn, within visible map bounds.

### Morning Wakeup Greetings
Timed full-screen animated greetings for each family member when they typically arrive in the kitchen for breakfast. Personalized per girl, with name and a fun message. Requires knowing each person's typical breakfast time and the tablet being awake.

### Flight Status on Travel Days
On days when Todd is traveling, show live flight status updates: on time / delayed / departed / landed. Requires RapidAPI key for AeroDataBox (scaffolded in code, key needed). Would show in the Daddy Countdown card with a live status badge.

### Family Travel Notifications
Expand the Daddy Countdown concept to track travel for other family members — not just Todd's FL↔WA trips. Would include a schedule editor so trips can be added without editing code directly.

### Blue Origin Launch Notifications
Show upcoming New Shepard / New Glenn launch schedules and countdowns. Alert the family when a launch is imminent. Potential data source: The Space Devs API (`thespacedevs.com`) — free tier available.

### Google Photos Integration
Pull random photos from a shared Google Photos album to display in the Daddy Countdown section or as a rotating background. Requires one-time OAuth login with Google (browser-based implicit flow, no backend server needed). Would replace or supplement the 14 hardcoded photos.

### Cargo Inference
Add "likely cargo" note to ship cards based on vessel type and route:
- Container ship from Shanghai → Mixed consumer goods / electronics
- Bulk carrier → Grain, coal, potash, or wood chips
- Tanker → Petroleum products
- RoRo → Vehicles (Glovis Star = Hyundai/Kia cars)

### Wake Lock API
Add `navigator.wakeLock.request('screen')` to keep the screen on while the app is active. Prevents the tablet from sleeping during normal display use without needing Developer Options.

---

## Version History

| Version | Changes |
|---------|---------|
| v5.3 | Whale cards sorted by distance from home, then recency; orca/whale card colors (black/gray) matching map icons |
| v5.2 | Whale sighting window extended to 72 hours; orca cards black, whale cards gray |
| v5.1 | Fixed Acartia species mapping (`type` field, not `profile`); geographic filter for Puget Sound; removed broken AeroDataBox calls |
| v5.0 | Fixed duplicate `data` variable JS crash in fetchWhales |
| v4.9 | Acartia.io whale API integrated with token; dual-format mapping (Acartia + Whale Museum); geographic bounding filter |
| v4.8 | Acartia API scaffolded; dad joke card includes link to orcanetwork.org; Whale Museum API identified as offline |
| v4.7 | Auto-refresh on deployment: detects new version from GitHub every 2 minutes and reloads |
| v4.6 | Namibia Fun Fact banner: SVG flag, 20 curated facts, slides up every 2 minutes, auto-dismisses |
| v4.5 | All flight times updated with exact schedules |
| v4.4 | Right panel layout overhauled: compact weather, whale section capped with scroll, daddy always visible |
| v4.3 | Blurred backdrop photo display (full photo shown uncropped) |
| v4.2 | Daddy countdown moved to right panel below whale sightings |
| v4.1 | Flight number inlined with arrival date/time |
| v4.0 | Exact WetzelBucks branding: white header, indigo `#4f46e5`, `#f7f8fa` background, system fonts |
| v3.9 | Dad Joke of the Day when no whale sightings |
| v3.8 | WetzelBoats rebrand |
| v3.7 | Rotating daddy photos (14 photos, 30s interval) |
| v3.6 | Fixed JS crash (unescaped apostrophe) |
| v3.5 | Daddy Countdown widget: photo, countdown, 3 states, Alaska Airlines flights |
| v3.0–3.4 | Wake line width, color, opacity, length iterations |
| v2.9 | All 4 boat icons unified viewBox |
| v2.5–2.8 | Wake line visibility fixes |
| v2.4 | Custom top-down SVG vessel icons rotate to AIS heading; whale silhouette icons |
| v2.3 | Single distance-sorted vessel list; scroll-to-card on hover |
| v2.2 | Photo flash fix; sticky stats bar |
| v2.1 | Map zoom/center from live browser (zoom 11, 47.3814, -122.4925) |
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
- **Static data lag** — Type, Flag, Destination fields take up to 15 minutes for new vessels
- **Acartia sightings without location** — pushed to bottom of whale list since distance can't be calculated
- **Flight times** — hardcoded; AeroDataBox integration scaffolded but requires RapidAPI key
- **Touch vs hover** — hover highlights work with mouse; on Pixel tablet, tap achieves same result
- **Auto-refresh** — requires `CURRENT_VERSION` bump with each deployment
- **Screen sleep** — Wake Lock API planned but not yet implemented; use Developer Options → Stay Awake While Charging as interim solution
- **No mobile layout** — designed but not yet built; current layout optimized for tablet landscape only
