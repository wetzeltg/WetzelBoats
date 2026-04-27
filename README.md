# Puget Sound Dashboard

A live maritime and wildlife dashboard for the southern Puget Sound, focused on the Port of Tacoma area. Built as a single-file web app deployable via GitHub Pages.

**Live URL:** `https://wetzeltg.github.io/puget_sound_dashboard/`

---

## What It Shows

### Vessel Traffic
- Live AIS (Automatic Identification System) feed of all commercial vessels in southern Puget Sound
- Vessels categorized as **Arriving**, **Departing**, **In Port** (docked at Port of Tacoma terminals), or **Anchored/Waiting** (at anchor in the water awaiting a berth)
- Ship cards show vessel name, photo (from MarineTraffic), type, flag, origin/destination, speed, and ETA
- All vessels sorted by distance from home (4910 La Hal Da Ave NE, Tacoma WA 98422)

### Interactive Map
- OpenStreetMap base layer centered on southern Puget Sound (zoom 11)
- Home location marker in the lower-right area of the default view
- Custom top-down vessel icons that **rotate to face the ship's actual heading**
- Custom whale silhouette icons for each species
- Click any map icon to open a popup with vessel/sighting details
- **Hover** over a map icon → highlights the corresponding card in the side panel (and vice versa)

### Whale & Orca Sightings
- Live sighting reports from The Whale Museum Hotline API
- Shows species, location, time reported, pod identity (for orcas), and individual count
- Distinguishes Southern Resident orcas (J, K, L pods) from Bigg's/Transient orcas

### Weather
- Real-time conditions at Port of Tacoma from Open-Meteo (no API key required)
- Temperature, wind speed/direction, humidity, barometric pressure, and conditions

---

## Map Icons

### Vessels (top-down view, rotates to live heading)
| Icon | Status | Color |
|------|--------|-------|
| Pointed bow, parallel sides, flat stern | Arriving | Blue |
| Same hull shape + 4-line wake | Departing | Green |
| Same hull shape + anchor symbol | In port (docked) | Amber |
| Same hull shape + faint wake + anchor | Anchored/waiting | Purple |

The anchor symbol consists of a ring, vertical shank, horizontal stock crossbar, and curved arms — the arc midpoint connects to the base of the shank.

### Marine Mammals (top-down silhouettes)
| Species | Appearance |
|---------|-----------|
| **Orca** | Black body, white belly patch, white eye patches, white saddle patch, tall straight dorsal fin |
| **Gray whale** | Blue-gray body, heavy mottling (barnacle blotches), no dorsal fin, knuckle bumps along dorsal ridge |
| **Humpback** | Dark green body, lighter belly, very long swept-back pectoral fins, small hooked dorsal fin set far back, knobby tubercles on rostrum |

All three whale species use the same traced silhouette outline and are distinguished by coloring only.

---

## Data Sources

| Data | Source | API Key Required |
|------|--------|-----------------|
| Live vessel positions & data | [AISStream.io](https://aisstream.io) WebSocket | Yes (free) |
| Weather | [Open-Meteo](https://open-meteo.com) | No |
| Whale sightings | [The Whale Museum Hotline](https://hotline.whalemuseum.org) | No |
| Ship photos | MarineTraffic (by MMSI) | No (public thumbnails) |
| Map tiles | OpenStreetMap via Leaflet.js | No |

---

## API Key Setup

The AISStream API key is hardcoded in `index.html`:

```javascript
let aisKey = localStorage.getItem('aisKey') || 'YOUR_KEY_HERE';
```

The key is free — sign up at [aisstream.io](https://aisstream.io). Since the repo is public and the key is free with no billing, this is acceptable for a personal project. If the key needs to be regenerated, log into AISStream and create a new one.

The ⚙ **API Keys** button in the top-right header can also be used to override the key via the browser's localStorage on any device.

---

## AIS Data Notes

The AIS stream sends two types of messages:

- **PositionReport** — arrives every few seconds per vessel: lat/lng, speed, heading, navigational status
- **ShipStaticData** — arrives every ~6 minutes per vessel: vessel name, type, destination, call sign, dimensions

Because static data arrives infrequently, ship cards may show `—` for Type, Flag, or Destination when first seen. These fields fill in automatically over 10–15 minutes as each vessel broadcasts its static update. The app also attempts to enrich vessel data via VesselFinder and MyShipTracking through a CORS proxy.

### Status Classification Logic
- **In port** — vessel is physically inside Port of Tacoma terminal boundaries (lat 47.24–47.30, lng -122.44 to -122.38) and speed < 1 knot, OR navigational status = moored/at anchor within port area
- **Anchored** — nav status = at anchor, OR speed < 1 knot outside port boundaries
- **Arriving** — destination includes "TACOMA" or heading southbound (135°–210°)
- **Departing** — destination is a non-Tacoma port, or heading northbound

### Bounding Box
The AIS stream is filtered to southern Puget Sound: `[[47.0, -122.8], [47.7, -122.2]]`

---

## Deployment

This is a **single-file static web app** — no server, no build step, no dependencies to install.

### GitHub Pages (current deployment)
1. Edit `index.html` directly in the GitHub web editor, or push via git
2. GitHub Pages auto-deploys within ~60 seconds of a commit
3. Hard refresh (`Ctrl+Shift+R`) on the live URL to bypass browser cache

### Running Locally
Just open `index.html` directly in Chrome. The AIS WebSocket, weather API, and whale API all work from a local file context.

### Updating on the Pixel Tablet (Washington)
The tablet opens the GitHub Pages URL in Chrome. After any update is deployed:
1. Open the dashboard in Chrome
2. Press the three-dot menu → Reload (or hard refresh)
3. The latest version loads automatically

To install as a home screen app on the Pixel tablet: Chrome menu → **Add to Home Screen**.

---

## Version History

| Version | Changes |
|---------|---------|
| v2.4 | Custom top-down vessel SVG icons rotate to live AIS heading; custom whale/orca silhouette icons (humpback, gray whale, orca) |
| v2.3 | Single unified vessel list sorted by distance from home; scroll-to-card on hover restored |
| v2.2 | Photo flash fix (noPhotoMMSI cache); sort by distance from home per section; sticky stats bar; no auto-scroll on hover |
| v2.1 | Map zoom/center set from live browser coordinates (zoom 11, centered at 47.3814, -122.4925) |
| v2.0 | Debug panel toggleable; map zoom corrected |
| v1.9 | Fixed duplicate `const anchored` JS crash |
| v1.8 | Tighter bounding box (southern Sound only); smarter status logic with anchored/waiting category; 5-stat strip |
| v1.7 | Vessel enrichment via VesselFinder/MyShipTracking; expanded flag table (150+ Maritime IDs); improved debug strip |
| v1.6 | Full MID flag table; vessel enrichment lookups; whale API URL fix |
| v1.5 | Fixed AIS WebSocket Blob parsing bug (was silently failing JSON.parse) |
| v1.4 | Version number added to header for deployment verification |
| v1.3 | Light theme (white background); all panels and cards restyled |
| v1.2 | Hover highlight system (card↔map icon bidirectional); proper ship/whale SVG icons |
| v1.1 | AIS key hardcoded; GitHub Pages deployment; demo data with realistic vessels |
| v1.0 | Initial build: map, weather, whale sightings, vessel cards, demo mode |

---

## File Structure

```
index.html        — entire app (HTML + CSS + JS, self-contained)
README.md         — this file
```

The app is intentionally kept as a single file for simplicity. All SVG icon definitions are embedded directly in the JavaScript as template literals.

---

## Known Limitations

- **Whale sighting photos** — not available; whale cards show species, location, and time only
- **Cargo data** — AIS does not transmit cargo contents; ship type gives a general indicator (container = mixed goods, bulk = grain/coal/potash, tanker = petroleum)
- **Vessel enrichment** — the VesselFinder/MyShipTracking lookups via CORS proxy are best-effort and may not always return data
- **Static data lag** — Type, Flag, and Destination fields may take up to 15 minutes to populate for newly-seen vessels
- **Whale API** — The Whale Museum Hotline (`hotline.whalemuseum.org`) occasionally goes offline; the app falls back to showing "No sightings reported in the last 24 hours"
