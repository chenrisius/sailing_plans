# NY Bay Sail Plan Generator

Automated sail-planning assistant for a **J/24 sloop** (24 ft, fin keel, 4 ft draft) operating in **Upper New York Bay**, departing from and returning to **Liberty Landing Marina** (Jersey City, NJ).

## Purpose

Given a requested time of day, the tool produces a complete, safety-vetted sail plan grounded in live marine data and the actual NOAA navigation chart. It removes the manual work of cross-checking forecasts, tides, currents, ferry timing, and chart hazards, and outputs deliverables ready to load onto a chartplotter and carry aboard.

## Deliverables

Each run produces:

1. **Conditions summary** — wind, tide, current, weather, sea state
2. **Sail plan** — route, per-leg headings/points of sail, ETAs, go/no-go and bailout criteria
3. **GPX route file** — waypoints + route in WGS84, loadable into a GPS/chartplotter
4. **One-page PDF reference sheet** — conditions, plan, verification table, and the route overlaid on the NOAA chart

## How It Works

**1. Sailing window.** The request maps to one of three windows, each with fixed cast-off, return-by, and bailout times:
- Morning (09:00–13:00)
- Afternoon (13:30–17:30)
- Evening (18:00–sunset+15 min)

Sunrise/sunset are computed locally from a solar-position formula.

**2. Live data pass** (bounded to ≤5 calls). All NOAA endpoints are pulled with `curl` (not cached fetchers):
- **Wind (live):** NDBC Station ROBN4, Robbins Reef — required cross-check against the forecast
- **Forecast:** NWS coastal-waters zone ANZ338, New York Harbor
- **Tides:** NOAA CO-OPS station 8518750, The Battery
- **Currents:** derived from the tide (slack ≈ HW/LW; ebb sets ~185°T, flood ~005°T)

Data-freshness gates apply (ROBN4 ob ≤2 h; forecast issuance ≤12 h). A stale forecast, missing tides, a non-full-res chart, or a no-sail wind/SCA/thunderstorm condition triggers a hard-stop that withholds the route.

**3. Chart-based validation.** The full-resolution NOAA Chart 12327 raster (6,074 × 8,000 px) is downloaded and verified (Statue of Liberty tan-patch cross-check). A pixel↔lat/lon transform validates every waypoint and leg for water depth (≥8 ft MLLW), land/shoal avoidance, and security-zone clearance.

## Safety Rules Enforced

- **Wind limits:** no-sail ≥25 kt sustained; reef suggestion when gusts reach 25; light-air/marginal flag <6 kt; jib ≥12 kt, genoa <12 kt
- **Boundaries:** all routes stay between the George Washington Bridge (N) and Verrazzano-Narrows Bridge (S)
- **Buttermilk Channel:** hard exclusion (no transit); its entrance buoy remains a legitimate mark
- **Security zones:** ≥0.10 nm clearance from Liberty and Ellis Islands (33 CFR 165.169)
- **Ferry traffic:** Anchorage Channel crossings timed into Staten Island Ferry gaps
- **No dead-run legs** (broad reach is the deepest downwind angle for a J/24)
- Total planned distance ≤85% of the theoretical maximum for the window

## Route Variety

The plan shape is chosen per run from wind direction, current, and time budget (compact Liberty loop, rounded loop, northbound Hudson leg, southbound toward the Verrazzano, extended SE-bay route, or a fresh shape), favoring the upwind/up-current leg first so the return is the easier point of sail. Recent plans are checked to avoid repetition.

## Reference Files

- `reference-track.gpx` — recorded ~11 nm reference sail through the Upper Bay
- `siferryschedule.pdf` — Staten Island Ferry timetable (verify against current NYC DOT schedule before relying on it)

## Output Naming

```
YYYY-MM-DD NYBay_{Morning|Afternoon|Evening}_Sail_Plan.gpx
YYYY-MM-DD NYBay_{Morning|Afternoon|Evening}_Sail_Plan.pdf
```
`YYYY-MM-DD` is the cast-off date in local time.
