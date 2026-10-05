<div align="center">

# 🌫️ vis-spirals

**Animated climate spirals of low-visibility (fog) hours from airport METAR observations**

[![CI](https://github.com/YOUR_USERNAME/vis-spirals/actions/workflows/ci.yml/badge.svg)](https://github.com/YOUR_USERNAME/vis-spirals/actions/workflows/ci.yml)
![Python](https://img.shields.io/badge/python-3.9%2B-blue)
![License: MIT](https://img.shields.io/badge/license-MIT-green)
![matplotlib](https://img.shields.io/badge/made%20with-matplotlib-11557c)

<img src="docs/animations/monthly_cumulative_spiral.gif" width="560" alt="Monthly cumulative hours with visibility below 1 km at LKNA">

</div>

---

How often is an airport fogged in, when in the year does it happen, and is it changing?
`vis-spirals` turns raw METAR/ASOS visibility reports into **polar "climate spiral" animations**
of the number of hours with visibility below 1 km. The calendar runs clockwise from January at
the top, so the seasonal fog cycle and year-to-year differences are visible at a glance.

The example animations use data from **LKNA – Náměšť nad Oslavou, Czech Republic**.

## Animations

### 1 · Monthly cumulative spiral
Running total of low-visibility hours within each year, month by month. Each year is coloured on
a gradient from oldest (purple) to newest (yellow): steep segments mean foggy months, and years
that end further from the centre had more fog overall.

<p align="center"><img src="docs/animations/monthly_cumulative_spiral.gif" width="520" alt="Monthly cumulative spiral"></p>

### 2 · Each year vs. the long-term mean
One frame per year. The red curve shows hours per day below 1 km; the grey curve is the
mean for that day of the year across the whole record.

<p align="center"><img src="docs/animations/annual_vs_climatology.gif" width="480" alt="Annual curve vs climatology"></p>

### 3 · Daily anomaly spiral
The departure of each day from its day-of-year mean, drawn day by day. Completed years fade to
grey, the current year is traced in red, and the yellow marker is the newest day. Points outside
the typical band were unusually foggy; points inside were unusually clear.

<p align="center"><img src="docs/animations/daily_anomaly_spiral.gif" width="520" alt="Daily anomaly spiral"></p>

## Quick start

```bash
git clone https://github.com/YOUR_USERNAME/vis-spirals.git
cd vis-spirals
pip install -e .

# put your station file in data/ (see data/README.md), then:
vis-spirals data/LKNA.csv -o docs/animations
```

This writes three GIFs:

| File | Animation | Frames |
|------|-----------|--------|
| `monthly_cumulative_spiral.gif` | Running yearly total, month by month | one per month |
| `annual_vs_climatology.gif` | Each year vs. the mean annual cycle | one per year |
| `daily_anomaly_spiral.gif` | Daily anomaly, drawn day by day | one per day (see `--anomaly-step`) |

### CLI options

```text
vis-spirals CSV [-s STATION] [-o OUT_DIR] [-a {annual,anomaly,cumulative} ...]
                [--threshold-km 1.0] [--anomaly-step 1] [--smooth-days 1] [--dpi 100]
```

| Option | Default | Description |
|--------|---------|-------------|
| `-s, --station` | file name | Station label used in titles |
| `-o, --out-dir` | `output/` | Where GIFs are written |
| `-a, --animations` | all | Render only selected animations |
| `--threshold-km` | `1.0` | Visibility threshold (e.g. `5` for mist, `0.55` for CAT I) |
| `--anomaly-step` | `1` | Days per frame in the anomaly spiral; `5` makes a 5× shorter GIF |
| `--smooth-days` | `1` | Rolling mean for the annual animation (e.g. `7`) |
| `--dpi` | `100` | Output resolution |

### Python API

```python
from vis_spirals import (
    load_observations, prepare_observations,
    daily_low_visibility, monthly_low_visibility,
    animate_monthly_cumulative_spiral, save_gif,
)

obs = prepare_observations(load_observations("data/LKNA.csv"), threshold_km=1.0)
monthly = monthly_low_visibility(obs)

anim = animate_monthly_cumulative_spiral(monthly, station="LKNA", cmap="viridis")
save_gif(anim, "my_spiral.gif", fps=10)
```

Works in Jupyter and Google Colab too — just `pip install git+https://github.com/YOUR_USERNAME/vis-spirals.git`.

## Data

The input is a CSV export from the
[Iowa Environmental Mesonet ASOS/METAR archive](https://mesonet.agron.iastate.edu/request/download.phtml),
which covers airports worldwide. Only two columns are needed:

| Column | Meaning |
|--------|---------|
| `valid` | observation time |
| `vsby` | visibility in statute miles (`M` = missing) |

See [`data/README.md`](data/README.md) for step-by-step download instructions.

## Method

1. Visibility is converted from miles to km (× 1.60934).
2. Each report is weighted by the **time elapsed since the previous report** (capped at 24 h),
   so hourly, half-hourly and irregular SPECI reports all yield true hours rather than report
   counts. Missing visibility is never counted as low.
3. Hours below the threshold are summed per day or per month.
4. The **climatology** is the mean for each day of the year (or month) over the full record;
   the **anomaly** is the departure from it. 29 February is dropped so every year fits the same
   365-day circle.
5. For the anomaly spiral, a constant offset keeps the radius positive; the dashed ring marks
   zero anomaly.

## Project structure

```
vis-spirals/
├── src/vis_spirals/
│   ├── data.py          # loading, time-weighting, daily/monthly aggregation
│   ├── animations.py    # the three polar animations
│   └── cli.py           # `vis-spirals` command
├── tests/               # pytest suite (synthetic data, no download needed)
├── docs/animations/     # GIFs embedded in this README
├── data/                # your station CSVs (git-ignored)
└── .github/workflows/   # CI: lint + tests on Python 3.9–3.12
```

## Development

```bash
pip install -e ".[dev]"
ruff check .
pytest
```

## Acknowledgements

- Observation data: [Iowa Environmental Mesonet](https://mesonet.agron.iastate.edu/), Iowa State University.
- Visual style inspired by Ed Hawkins' climate spirals.

## License

[MIT](LICENSE)
