# TD4: CO2 time series — collection, cleaning, smoothing, projection

Real atmospheric CO2 (ppm) for Le Mans, pulled from Open-Meteo's Air Quality API (a
reanalysis+forecast model, not a single sensor), cleaned, smoothed two ways, and
projected 7 days forward with an honesty note about the limits of that projection.

- `scripts/fetch_series.py`: pulls 90 days of real hourly CO2, aggregates to daily
- `scripts/analyze_series.py`: reindexes to every calendar day (flags gaps), computes
  a 7-day rolling mean, exponential smoothing (alpha=0.3), and a linear-trend
  projection; writes `docs/data/series_analysis.json`
- `.github/workflows/update_series.yml`: runs weekly (Monday 06:00 UTC), pulls a
  fresh 90-day window each time, and commits the result
- `docs/index.html`: GitHub Pages dashboard — 4 KPI cards, the honesty note, and a
  4-series chart (observed, rolling mean, exponential smoothing, projection)

Real run (2026-11-03): 90 days, 0 gaps, trend +0.108 ppm/day, most recent reading
447.5 ppm.
