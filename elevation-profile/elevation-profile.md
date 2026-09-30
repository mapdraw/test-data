# Elevation Profile

Manual test of the elevation profile summary (ascent, descent, highest/lowest point, hiking time) for the sources File, Google and GeoAdmin. Three tracks are Strava GPX exports (dense GPS, ~0.6–7 m between points); `vogesenkammweg` is a planned route in the Vosges (France) from ich-geh-wandern.de (~86 m between points on average; GeoAdmin returned no profile for it). `mapdraw-route-no-elevation` is a route exported from MapDraw without elevation (~19 m between points on average).

One folder per track: the GPX, one console snapshot per app behavior (`YYYY-MM-DD-<behavior>.json`) and, where available, `map.geo.admin.ch-profile.csv` (profile of the same GPX exported from map.geo.admin.ch) and `swisstopo-app.gpx` (the track as returned by the swisstopo app share link). `issue-geoadmin-web-mapviewer` holds the text of the issue posted from these results.

## Snapshot

With the default settings (Elevation Provider Google, Prefer file elevation data checked), import the GPX and open the profile (Source: File, or Google if the GPX has no elevation), then in the console:

```js
window.snaps = [];
```

Run once per source:

```js
snaps.push({
  source: currentSource,
  summary: document.getElementById("d3-summary-html").innerText.replace(/\s+/g, " ").trim(),
  points: currentRawData.map((d) => [+d.distance.toFixed(1), d.elevation]),
});
```

- File: as imported (skip if the GPX has no elevation)
- Google: click ⊗ next to "Source: File" (already shown if the GPX has no elevation)
- GeoAdmin: switch Elevation Provider in Settings, select the track and reopen the profile

Then `copy(JSON.stringify(snaps))` and save in the track folder. Never click ⊕: it writes the API elevation into the track.

Snapshots are stored as recorded. The `vogesenkammweg` every-point and 25m-sampling snapshots end with an empty entry (`"source":null`), recorded after the GeoAdmin attempt returned no profile.

`monte-gambarogno/2026-09-30-sample-distances.json` was recorded to verify MapDraw commit fbb5801 (API samples placed at their recorded path distance instead of searched for on the track) and equals the 25m-sampling snapshot exactly. Comparing it with a GPX exported after ⊕ showed that the fixed code writes all 584 samples of both APIs to within 1 cm of their API value, where the previous code was off by up to 2.6 m.

## Results

Ascent / descent in m. `every-point` = live app before the changes of 2026-09-28 (every point sent; Google resampled to 5000 points for longer paths; for mapdraw-route-no-elevation measured by disabling the sampling in the console); `25m-sampling` = one point every 25 m (`ELEVATION_SAMPLE_SPACING` in `js/elevation.js`); `25m-sampling-keep-vertices` = same, but a path already sparser than 25 m keeps its vertices (only affects vogesenkammweg, where it equals every-point); `25m-sampling-file` = the File source is sampled the same way (API rows unchanged).

| Track                      | Source           | every-point           | 25m-sampling          | 25m-sampling-file |
| :------------------------- | :--------------- | :-------------------- | :-------------------- | :---------------- |
| beinwil-reh-passwang       | File             | 940 / 546, 4h40       | same                  | 894 / 500, 4h26   |
|                            | Google           | 1328 / 929, 5h59      | 1095 / 695, 5h08      |                   |
|                            | GeoAdmin         | 1774 / 1378, 8h23     | 1048 / 652, 4h57      |                   |
|                            | map.geo.admin.ch | 1774 / 1378           |                       |                   |
|                            | swisstopo app    | 881 / 487, 4h22       |                       |                   |
| monte-gambarogno           | File             | 1038 / 1037, 5h38     | same                  | 1004 / 1003, 5h22 |
|                            | Google           | 1784 / 1785, 8h21     | 1278 / 1279, 6h25     |                   |
|                            | GeoAdmin         | 2907 / 2907, 14h18    | 1241 / 1242, 6h13     |                   |
|                            | map.geo.admin.ch | 2895 / 2895           |                       |                   |
|                            | swisstopo app    | 1000 / 999, 5h16      |                       |                   |
| basel-winterthur           | File             | 1580 / 1403           | same                  | 1506 / 1329       |
|                            | Google           | 1464 / 1284           | 1412 / 1232           |                   |
|                            | GeoAdmin         | 1300 / 1123           | 1019 / 842            |                   |
|                            | map.geo.admin.ch | 1299 / 1122           |                       |                   |
| vogesenkammweg             | File             | 19180 / 18939, 130h17 | same                  | same              |
|                            | Google           | 18152 / 17921, 127h02 | 17717 / 17487, 125h44 |                   |
| mapdraw-route-no-elevation | Google           | 882 / 730, 6h24       | 909 / 756, 6h21       |                   |
|                            | GeoAdmin         |                       | 635 / 500, 5h35       |                   |
|                            | map.geo.admin.ch | 645 / 510             |                       |                   |
|                            | swisstopo app    | 899 / 713, 6h22       |                       |                   |

Every-point GeoAdmin matches map.geo.admin.ch within 12 m. On the three Strava tracks, 25 m sampling lowers ascent for Google and GeoAdmin, while highest/lowest points change by at most 2 m. On `vogesenkammweg` it lowered Google ascent by ~2%; keep-vertices gives exactly the every-point values again. Sampling the File source too brings it close to the swisstopo app (Beinwil 894 vs 881 m, Gambarogno 1004 vs 1000 m). On `mapdraw-route-no-elevation` Google gives far more ascent than GeoAdmin: over the same 833 points its heights change direction 302 times vs 79, and at 100 m resolution the two give 652 vs 605 m.

swisstopo app: numbers from its share pages (Beinwil https://swisstopo.app/i/5/GE3EVMHV, mapdraw-route-no-elevation https://swisstopo.app/i/5/S3GNCAG8) and, for Gambarogno, from a screenshot of the Android app 1.25.0 (not stored); `swisstopo-app.gpx` from its share API. For Beinwil, Douglas-Peucker with ~3 m tolerance plus the Swiss hiking formula, applied to the heights in its `swisstopo-app.gpx`, reproduces its numbers exactly (reconstructed, not from source).
