# Google Health live fixes

This branch is based on upstream PR #67 (`fix: implement Google Health API mappings for all data types`) and adds small production-hardening changes that were validated on a live Google Health + InfluxDB 1.x + Grafana stack.

## Why this branch exists

Upstream `main` documents a broad Fitbit/Grafana dashboard, but in `HEALTH_API_PROVIDER=google` mode several collectors still skipped Google Health data with `mapping not finalized yet` warnings. This made the bundled dashboard look much emptier than the README suggested.

## Existing upstream work checked

- PR #67 (`ZeSlammy/pr/google-full-mapping`) is the best current base. It maps Google Health sleep, HRV, breathing rate, skin temperature, SPO2, resting HR, activity totals and HR zones into the existing InfluxDB schema, and adds pagination for intraday responses.
- PR #72 covers similar mapping work but was closed and its source branch/repo was no longer fetchable during inspection.
- PR #73 adds Google battery support, but battery is not needed for this deployment and is not a blocker.
- PR #70 fixes sleep-day anchoring, but its branch is older than #67 and cannot be applied wholesale without reverting the fuller Google mapping/pagination changes. It should be revisited as a focused follow-up.
- PR #76 handles corrupted token JSON and is orthogonal to the Google mapping problem.

## Additional fixes on this branch

- Scheduled token refresh now updates the global in-memory `ACCESS_TOKEN` instead of only refreshing the token file.
- Google mode skips the Fitbit-only battery update at startup and in schedules.
- Google mode uses a 15-minute intraday schedule instead of the legacy 3-minute Fitbit schedule to reduce noisy API churn.

## Battery

Battery is intentionally treated as low priority. Google Health does not expose the same device battery measurement used by the old Fitbit API path, and the dashboard should not depend on it.

## Validation pattern

1. `python3 -m py_compile Fitbit_Fetch.py`
2. Run against a protected Google Health token on the target stack.
3. Confirm collector logs show writes for HeartRate_Intraday, Steps_Intraday, Sleep Summary/Levels, HRV, BreathingRate, SPO2_Intraday, Skin Temperature Variation, RestingHR, Activity Minutes/Steps/Distance/Calories/HR zones.
4. Verify Grafana dashboard queries against InfluxDB with a concrete time range.
