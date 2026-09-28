Posted as https://github.com/geoadmin/web-mapviewer/issues/1599 with `monte-gambarogno.zip` (= `../monte-gambarogno/12112256082.gpx`), a swisstopo app screenshot and `../monte-gambarogno/map.geo.admin.ch-profile.csv` attached.

---

# Elevation profile of an imported GPX track: ascent and hiking time far too high because the GPX simplification is never applied

## Steps to reproduce

1. Open https://map.geo.admin.ch
2. Import the attached `monte-gambarogno.gpx` (in `monte-gambarogno.zip`; Strava export of a 14.6 km hike, 24,808 track points, about 0.6 m apart)
3. Select the track and open its elevation profile

## Result

Ascent 2895 m, descent 2895 m, hiking time 14 h 13 min (CSV export of the profile attached).

## Expected

The GPX file contains 1038 m of ascent. The swisstopo app (Android 1.25.0) shows 1000 m / 999 m and 5 h 16 min for the same file (screenshot attached).

## Cause

`ShowGeometryProfileButton.vue` dispatches `setProfileFeature` with `simplifyGeometry = true` for GPX layers (comment: "PB-800 : to avoid a coastline paradox we simplify the geometry of GPXs"), and `profile.store.js` stores it in `state.simplifyGeometry`. But nothing reads that state: `InfoboxContent.vue` renders `<GeoadminElevationProfile :points="currentProfileCoordinates" …>` without the `simplify` prop, which defaults to `false` in `GeoadminElevationProfile.vue`. So all 24,808 points are sent to `profile.json` and the profile has one point per GPS point (24,808 rows in the CSV). `totalAscent()` and `hikingTime()` in `utils.ts` then sum up every small height difference caused by GPS scatter.

## Suggested fix

Pass the stored flag to the component (`:simplify="…"`), so that `GeoadminElevationProfile.vue` simplifies the track with `GEOMETRY_SIMPLIFICATION_TOLERANCE` (12.5, `config.ts`) before requesting the profile.

For reference: simplifying this track with Douglas-Peucker at 12.5 m in LV95 leaves 153 points; the profile `profile.json` returns for them gives 1067 m ascent, 1067 m descent and 5 h 12 min with the formulas from `utils.ts`.
