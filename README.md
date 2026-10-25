# Garmin Health Signals

A local dashboard for exploring your Garmin health readings, personal ranges,
and patterns across several days. A Next.js frontend calls a Python adapter that
retrieves data through the Garmin Personal API.

Insights are descriptive training context, **not medical advice or diagnoses**.
An unusual reading does not automatically mean something is wrong, and
measurements that change together do not establish a cause.

## Requirements

- Python 3.10 or newer.
- Node.js and npm compatible with the Next.js version in
  [frontend/package.json](frontend/package.json).
- A Garmin Personal API key with access to your account's data.
- An internet connection for Garmin requests.

## Getting started

The following commands use Windows PowerShell, starting at the repository root.

### 1. Install Python dependencies

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

### 2. Configure your API key

Create a `.env` file in the repository root containing:

```dotenv
GARMIN_PERSONAL_API_KEY=your_personal_api_key
```

The Python scripts also accept this key from the process environment.
`GARMIN_API_KEY` is supported as an alternative name.

Never commit or share your key. The root `.env` file is ignored by Git.

### 3. Start the dashboard

```powershell
Set-Location frontend
npm install
npm run dev
```

Open <http://127.0.0.1:3000>.

The server expects the Python interpreter in the root `.venv` directory. If your
setup differs, set these environment variables before starting Next.js:

| Variable | Purpose |
| --- | --- |
| `GARMIN_PROJECT_ROOT` | Absolute repository-root path. Defaults to the parent of the frontend working directory. |
| `GARMIN_PYTHON_EXECUTABLE` | Absolute Python executable path. Defaults to the repository's virtual environment interpreter. |

On non-Windows systems, use `.venv/bin/python` instead of
`.\.venv\Scripts\python.exe`.

## Using the dashboard

- **Refresh data** retrieves fresh readings from Garmin. It is not a cached refresh.
- **7D / 30D** changes the chart and pattern view using the loaded 30-day dataset;
  it does not trigger another Garmin request.
- **What's different** rows expand into short explanations with the reading's
  date and comparison range.
- **Patterns worth noticing** shows current and historical observations.
  Expand **Why this appeared** for dates and supporting evidence.
- The recovery timeline brings HRV, sleep duration and score, resting heart
  rate, Body Battery, and stress together.

Unavailable measurements stay unavailable rather than becoming zero. A missing
signal does not mean that signal was normal.

## How comparisons work

The dashboard uses personal history rather than population targets:

- Personal ranges use the 10th to 90th percentiles of available readings in the
  preceding 28 days, requiring at least 14 readings.
- HRV uses Garmin's balanced range when available.
- Historical pattern checks use each day's own Garmin bounds or earlier
  readings, never future measurements.
- The dashboard fetches 28 extra days for pattern baseline context, while
  keeping displayed metric series inside the requested period.
- Sleep duration and sleep score count as one warning group in training guidance,
  not two independent warnings.

### Pattern rules

These are transparent product heuristics, **not clinically validated thresholds**.
Patterns appear only when their evidence requirements are met.

| Pattern | Detection rule |
| --- | --- |
| Sustained lower HRV | At least 3 consecutive days below the dated HRV comparison range. |
| Combined recovery change | At least 3 consecutive days with both low HRV and high resting heart rate. |
| Repeated shorter sleep | At least 4 shorter-than-usual nights in 5 fully measured calendar nights. |
| Sustained higher resting heart rate | At least 3 consecutive days above its earlier personal range. A covering combined recovery event replaces this duplicate warning. |
| HRV moving toward usual | Three low-HRV days followed by 3 strictly rising readings, closing at least 20% of the gap to a fixed earlier range without exceeding its upper bound. |
| A steadier stretch | All 7 calendar days have HRV, sleep duration, and resting heart rate within their dated ranges. |
| Sleep and HRV connection | At least 14 paired dates in a 28-day window, including 4 shorter and 4 usual nights. Low HRV occurs on at least 75% of shorter nights and at least 50 percentage points more often than after usual nights. |
| Return after a busier exercise week | Three days of low HRV and high resting heart rate followed by 2 in-range days, plus an exercise week over twice the preceding two-week average. Both comparison weeks must contain recorded activities. |

Missing days break consecutive-day streaks. Overlapping detections of the same
type are consolidated, retaining the latest supporting window and its evidence.
Past observations are labeled separately and do not give advice for today's
workout. A positive pattern does not guarantee health or training readiness.
