# Elevation Profile

Manual test of the elevation profile summary (ascent, descent, highest/lowest point, hiking time) for the sources File, Google and GeoAdmin. Three tracks are Strava GPX exports (dense GPS, ~1–7 m between points); `vogesenkammweg` is a planned route from ich-geh-wandern.de (~86 m between points, outside GeoAdmin coverage).

One folder per track: the GPX, `map.geo.admin.ch-profile.csv` (profile of the same GPX exported from map.geo.admin.ch) and one console snapshot per app behavior, named `YYYY-MM-DD-<behavior>.json`.

## Snapshot

Import the GPX, open the profile (Source: File), then in the console:

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

- File: as imported
- Google: click ⊗ next to "Source: File"
- GeoAdmin: switch Elevation Provider in Settings, reopen the profile

Then `copy(JSON.stringify(snaps))` and save in the track folder. Never click ⊕: it writes the API elevation into the track.

## Results

Ascent / descent in m. `every-point` = live app before 2026-09-28 (every point sent, Google capped at 5000); `25m-sampling` = one point every 25 m (`ELEVATION_SAMPLE_SPACING` in `js/elevation.js`); `25m-sampling-keep-vertices` = same, but a path already sparser than 25 m keeps its vertices (only affects vogesenkammweg, where it equals every-point); `25m-sampling-file` = the File source is sampled the same way (API rows unchanged).

| Track                | Source           | every-point           | 25m-sampling          | 25m-sampling-file |
| :------------------- | :--------------- | :-------------------- | :-------------------- | :---------------- |
| beinwil-reh-passwang | File             | 940 / 546, 4h40       | same                  | 894 / 500, 4h26   |
|                      | Google           | 1328 / 929, 5h59      | 1095 / 695, 5h08      |                   |
|                      | GeoAdmin         | 1774 / 1378, 8h23     | 1048 / 652, 4h57      |                   |
|                      | map.geo.admin.ch | 1774 / 1378           |                       |                   |
|                      | swisstopo app    | 881 / 487, 4h22       |                       |                   |
| monte-gambarogno     | File             | 1038 / 1037, 5h38     | same                  | 1004 / 1003, 5h22 |
|                      | Google           | 1784 / 1785, 8h21     | 1278 / 1279, 6h25     |                   |
|                      | GeoAdmin         | 2907 / 2907, 14h18    | 1241 / 1242, 6h13     |                   |
|                      | map.geo.admin.ch | 2895 / 2895           |                       |                   |
| basel-winterthur     | File             | 1580 / 1403           | same                  | 1506 / 1329       |
|                      | Google           | 1464 / 1284           | 1412 / 1232           |                   |
|                      | GeoAdmin         | 1300 / 1123           | 1019 / 842            |                   |
|                      | map.geo.admin.ch | 1299 / 1122           |                       |                   |
| vogesenkammweg       | File             | 19180 / 18939, 130h17 | same                  | same              |
|                      | Google           | 18152 / 17921, 127h02 | 17717 / 17487, 125h44 |                   |

Every-point GeoAdmin equals map.geo.admin.ch. On dense GPS tracks, 25 m sampling removes the jitter noise; highest/lowest points are unchanged. On the sparse planned route it lost ~2% ascent because even samples miss the vertices; keep-vertices fixes that. Sampling the File source too brings it in line with the APIs and near the swisstopo app.

swisstopo app (Beinwil): numbers from its share page https://swisstopo.app/i/5/GE3EVMHV, `swisstopo-app.gpx` from its share API. It keeps the file elevation; Douglas-Peucker with ~3 m tolerance plus the Swiss hiking formula reproduces its numbers exactly (reconstructed, not from source).
