# REFLECTIONS (TD4)

## 1. Catching the wrong "no real data" assumption
The concrete check would have been to actually read Open-Meteo's Air Quality API docs (or just call it once with `past_days=90` and look at the response) before deciding historical data wasn't available.

## 2. Prediction: rising, falling, or flat?
Before running anything, I'd predict **rising**: these 90 days span early August to early November, which is the end of the Northern Hemisphere's growing season moving into autumn, when plants are absorbing less CO2 through photosynthesis. Checking the real output confirms it — trend slope +0.108 ppm/day, rising.

## 3. Does 0 gaps mean the gap-detection code is unnecessary?
No. Zero gaps in my run tells me this specific 90-day pull from this specific source was clean — not that the source is guaranteed to always be clean.

## 4. Rolling mean vs. exponential smoothing — a real situation for each
From my own output, the last 5 rolling-mean values (443.86 → 448.81) lag behind the last 5 exponentially-smoothed values (446.41 → 448.27) during the rise. I'd prefer the **rolling mean** when I want a stable read that doesn't overreact to one noisy day — e.g. reporting "typical CO2 this week" without one odd reading swinging the number. I'd prefer **exponential smoothing** when a real shift just started and I want the smoothed line to catch up to it quickly — e.g. noticing a sudden rise from a nearby wildfire or a local industrial event, where a 7-day flat average would hide the change for most of a week.

## 5. What happens to the data files if the API call fails every retry?
Looking at the workflow: `fetch_series.py` would raise `RuntimeError` after 4 failed attempts, which crashes the job before `analyze_series.py` even runs. The workflow has no step that writes placeholder data on failure, and the `git commit`/`git push` steps never execute, so nothing is overwritten.

## 6. How many days would you personally trust this projection?
Based on my own slope (+0.108 ppm/day) and projection (446.17 → 446.82 ppm over 7 days), I'd trust it for about **3-4 days**, not the full 7. The honesty note is right that a 90-day window can't be extrapolated across a season boundary, and a slope this shallow (about 0.1 ppm/day) is close enough to noise that a few consecutive unusual days could flip its sign — I'd want to see the rolling mean actually continue climbing for another few days before trusting the straight line much further out.

## 7. Why did changing location/variable need no other code changes?
`analyze_series.py`'s functions never mention CO2, Le Mans, or any specific variable by name — `clean_series()`, `rolling_mean()`, `exponential_smoothing()`, and `linear_trend_projection()` all operate on a generic `{"date": ..., "value": ...}` structure, where `value` could be any numeric series. The only place a variable name or location appears at all is in `fetch_series.py`'s `LAT`, `LON`, and the `hourly` parameter string — the analysis logic was written against the shape of the data, not its meaning, so it stayed correct automatically.
