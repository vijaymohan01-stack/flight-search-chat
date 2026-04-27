# Flight Search Chat — Session Context

## Project Overview
Chat-style flight search UI powered by Google Flights data (via the [fli](https://github.com/punitarani/fli) library's reverse-engineered API). Integrated as a new tab in Vijay HQ.

## Architecture

### Frontend (`flight-search-chat/`)
- **Repo**: `vijaymohan01-stack/flight-search-chat` (GitHub)
- Static HTML/CSS/JS chat interface with:
  - Quick form (From/To/Date/Class/Stops)
  - Quick chip shortcuts (SFO→JFK, LAX→LHR, etc.)
  - Chat bubble UI with animated flight result cards
  - Orange accent color (#f97316) matching Vijay HQ card-7 theme
- **API endpoints**: Tries localhost:5004 first, falls back to `flights-api.platonicsundry.com`
- **Netlify URL**: `vijay-flight-search.netlify.app` (NOT YET DEPLOYED — see remaining work)

### Backend (`~/.local/flask-servers/flight-search/server.py`)
- Flask server on port **5004**
- Ports fli's Google Flights API logic directly (no Python 3.10+ dependency)
- Uses `curl_cffi` for Chrome TLS impersonation
- Endpoints:
  - `POST /api/search-flights` — search flights with origin/destination/date/class/stops
  - `GET /api/health` — health check
- **Verified working**: SFO→JFK returns real flight results ($259 Alaska, $259 JetBlue, etc.)

### Infrastructure
- **Cloudflare Tunnel**: `flights-api.platonicsundry.com` → localhost:5004 (DNS CNAME added, config updated)
- **Flask orchestration**: Added to `~/.local/flask-servers/start-flask-servers.sh`
- **Tunnel verified**: `curl https://flights-api.platonicsundry.com/api/health` → `{"status": "ok"}`

### Vijay HQ Integration (pushed to GitHub)
- New "Flight Search" card (#7) with airplane icon on home panel
- New `panel-flights` iframe section
- New "Flights" tab in bottom tab bar with orange accent
- Config.json updated with `flight-search` project entry
- CSS: `--color-card-7: #f97316` (orange) with full hover/active/animation styles

### Forked Repository
- `vijaymohan01-stack/fli` — forked from `punitarani/fli`
- Cloned to `/Users/vijaymohan/Documents/Projects/fli`
- Note: Cannot install directly on Mac Mini (requires Python 3.10+, system has 3.9.6)
- The Flask server ports the relevant logic directly instead

## Current Status

### ✅ Completed
- [x] Fork `punitarani/fli` → `vijaymohan01-stack/fli`
- [x] Clone to local
- [x] Build Flask API wrapper with correct Google Flights request format
- [x] Test API — returns real flight data
- [x] Build premium chat UI (index.html, style.css)
- [x] Create GitHub repo `vijaymohan01-stack/flight-search-chat`
- [x] Push to GitHub
- [x] Integrate into Vijay HQ (card, panel, tab, CSS, config)
- [x] Push Vijay HQ changes to GitHub
- [x] Add Cloudflare Tunnel DNS route (`flights-api.platonicsundry.com`)
- [x] Update `start-flask-servers.sh`
- [x] Restart cloudflared with new config
- [x] Verify tunnel health check works

### ❌ Remaining
- [ ] **Deploy `flight-search-chat` to Netlify as `vijay-flight-search`**
  - NPM cache is corrupted with root-owned files
  - Fix: `chown -R $(whoami) ~/.npm && npm cache verify`
  - Then: `npx netlify-cli deploy --prod --dir .` (from flight-search-chat dir)
  - Or: manually link the GitHub repo via Netlify web UI
- [ ] Write session context for fli project (optional)
- [ ] Restart flask-servers via launchctl to persist the new flight-search server

## Key Files
| File | Purpose |
|------|---------|
| `~/.local/flask-servers/flight-search/server.py` | Flask API wrapping Google Flights |
| `~/Documents/Projects/flight-search-chat/index.html` | Chat UI frontend |
| `~/Documents/Projects/flight-search-chat/style.css` | Premium dark theme CSS |
| `~/Documents/Projects/vijay-hub/index.html` | Vijay HQ with new Flights tab |
| `~/Documents/Projects/vijay-hub/style.css` | HQ styles with card-7 orange accent |
| `~/Documents/Projects/vijay-hub/config.json` | HQ config with flight-search entry |
| `~/.cloudflared/config.yml` | Tunnel config with flights-api route |
| `~/.local/flask-servers/start-flask-servers.sh` | Flask server orchestration |

## Technical Notes
- Google Flights API requires browser TLS fingerprinting — `curl_cffi` with `impersonate="chrome"` handles this
- Airport arrays must use exactly 3 levels of nesting: `[[[code, 0]]]` (not 4!)
- The API URL is: `https://www.google.com/_/FlightsFrontendUi/data/travel.frontend.flights.FlightsFrontendService/GetShoppingResults`
- Response is JSONP-prefixed with `)]}'` — must be stripped before JSON parsing
- Flight data lives at `parsed[0][2]` → `json.loads()` → indices [2] and [3] contain flight arrays
