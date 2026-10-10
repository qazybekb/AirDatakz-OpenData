# AirData.kz — Open Air Quality Data for Kazakhstan

Cleaned, quality-checked air quality measurements from government and independent monitoring networks across Kazakhstan. Free to use for research, journalism, education, and public awareness.

**Website:** [airdata.kz](https://airdata.kz) · **Contact:** airdatakz@gmail.com · **Updated daily**

---

## Additional Data (Google Drive)

Larger historical datasets that don't fit in this repository are available on Google Drive:

**[AirData.kz — Historical Archives](https://drive.google.com/drive/folders/1M_GBxFrVUxeL0DsVgCPpxjALyS-MlZfM?usp=share_link)**

| Dataset | Description | Period | Size |
|:--------|:------------|:-------|:-----|
| **AirKaz** | Daily PM2.5 from 41 low-cost sensors across Almaty. Per-station readings with coordinates. | 2017–2020 | ~1.2 MB |
| **EcoGosFond.kz** | Government environmental reports from ecogosfond.kz — annual, quarterly, and monthly pollution statistics for all of Kazakhstan. PDF and original DOC formats. | 2005–2022 | ~2.2 GB |
| **KGMT Historical** | Hourly air quality data extracted from KazHydroMet Excel archives. 17 cities including Aktau, Aktobe, Atyrau, Karaganda, Kostanay, Pavlodar, Shymkent, and more. CSV format. | 2018–2022 | ~777 MB |

---

## Directory Structure

```
csv/
├── almaty/                     Almaty — 9 parameters, 2017–present
│   ├── pm25.csv.gz                625K hourly readings
│   ├── pm10.csv.gz                430K
│   ├── co.csv.gz                  452K
│   ├── no2.csv.gz                 445K
│   ├── no.csv.gz                  419K
│   ├── so2.csv.gz                 374K
│   ├── tsp.csv.gz                 118K
│   ├── h2s.csv.gz                   4K
│   ├── o3.csv.gz                  432
│   └── daily/                  Daily city-wide summaries
│       ├── pm25.csv.gz            3,284 days
│       └── ...
│
├── astana/                     Astana — 11 parameters, 2018–present
│   ├── pm25.csv.gz                267K hourly readings
│   ├── ...
│   └── daily/
│
├── karaganda/                  Karaganda region (KazHydroMet monitors in Karaganda, Temirtau, Saran, Balkhash, Zhezkazgan) — 12 parameters, 2018–present
│   ├── pm25.csv.gz                123K hourly readings
│   ├── ...                        (includes CH₄, THC, NH₃)
│   └── daily/
│
├── rest_of_kz/                 All other Kazakhstan cities — 9 parameters, 2020–present
│   ├── pm2_5.csv.gz               4.3M hourly readings
│   ├── ...                        (KazHydroMet government stations)
│   └── daily/
│
├── stations.csv                Station registry: id, name, city, coordinates, source, operator
└── stations.geojson            Same registry as GeoJSON points (WGS84)
```

---

## File Formats

All files are **gzip-compressed CSV** with UTF-8 encoding and header row.

### Hourly files (`{city}/{parameter}.csv.gz`)

One row per station per hour. Only measurements that passed all quality checks are included.

| Column | Type | Description |
|:-------|:-----|:------------|
| `datetime_utc` | timestamp | Start of the measured hour in UTC, e.g. `2026-10-08 15:00:00+00:00` (all sources) |
| `station_id` | string | Unique station identifier |
| `station_name` | string | Station name / location |
| `source` | string | Data source (see Sources below) |
| `lat` | float | Station latitude (WGS84) |
| `lon` | float | Station longitude (WGS84) |
| `value_ugm3` | float | Measured value in harmonized units (see Parameters) |
| `raw_value` | float | Original value before unit conversion |
| `raw_unit` | string | Original unit from the source |
| `conversion_note` | string | What conversion was applied |
| `cluster_id` | integer | Geographic cluster assignment |
| `cluster_name` | string | Cluster name (e.g., "City Center", "North Central") |

### Daily files (`{city}/daily/{parameter}.csv.gz`)

One row per day. City-wide average computed from geographic cluster averages, ensuring no single dense cluster dominates.

| Column | Type | Description |
|:-------|:-----|:------------|
| `date` | date | Calendar date |
| `value_ugm3` | float | City-wide daily average |
| `median_value` | float | Median of cluster daily averages |
| `min_cluster` | float | Lowest cluster average that day |
| `max_cluster` | float | Highest cluster average that day |
| `std_cluster` | float | Standard deviation across clusters |
| `n_clusters` | integer | Number of reporting clusters |
| `n_stations` | integer | Total reporting stations |

### `rest_of_kz` files

Hourly (`rest_of_kz/{parameter}.csv.gz`), one row per station per hour, clean values only:
`datetime_utc`, `station_id`, `station_name`, `city`, `lat`, `lon`, `value_ugm3`, `raw_value`, `raw_unit`
(`station_name`, `city`, `lat`, `lon` are empty for stations we have no metadata for).

Daily (`rest_of_kz/daily/{parameter}.csv.gz`), one row per city per day:
`date`, `city`, `avg_ugm3` (mean of the clean hourly values of the city's stations), `n_stations`.

---

## Parameters

| Code | Parameter | Unit | Notes |
|:-----|:----------|:-----|:------|
| `pm25` | PM2.5 — fine particulate matter | µg/m³ | Primary health indicator. Longest history. |
| `pm10` | PM10 — coarse particulate matter | µg/m³ | |
| `co` | Carbon monoxide | µg/m³ | |
| `no2` | Nitrogen dioxide | µg/m³ | |
| `no` | Nitric oxide | µg/m³ | |
| `so2` | Sulfur dioxide | µg/m³ | |
| `o3` | Ozone | µg/m³ | Limited stations |
| `h2s` | Hydrogen sulfide | µg/m³ | |
| `tsp` | Total suspended particles | µg/m³ | KGMT only |
| `nh3` | Ammonia | µg/m³ | Astana, Karaganda |
| `ch4` | Methane | µg/m³ | Karaganda only |
| `thc` | Total hydrocarbons | µg/m³ | Karaganda only |
| `pm2_5` | PM2.5 (national naming) | µg/m³ | rest_of_kz uses this code |
| `pmtot` | Total PM (national naming) | µg/m³ | rest_of_kz only |

---

## Data Sources

| Source | Type | Stations | Cities |
|:-------|:-----|:---------|:-------|
| `kgmt` | Government reference-grade (KazHydroMet) | 141+ | All Kazakhstan |
| `openaq` | International aggregator (OpenAQ) | 22 | Almaty, Astana |
| `waqi` | International aggregator (aqicn.org) | 11 | Almaty |
| `airgradient` | Low-cost sensors (AirGradient) | 139 | Almaty |
| `airkaz` | Low-cost sensors, historical (AirKaz) | 41 | Almaty (2017–2020) |

---

## Data Quality

Every measurement passes automated cleaning rules before inclusion.

**City files (`almaty`, `astana`, `karaganda`)**

| Rule | Check | What it catches |
|:-----|:------|:----------------|
| Range | Negative or missing values | Impossible values |
| Hard cap | Value at or above a physical limit (e.g. PM2.5 ≥ 1,000 µg/m³) | Implausible readings, instrument ceilings |
| Constant station | One value makes up ≥ 70% of a station-month | Frozen or dead instruments |
| Stuck sensor | Identical value for 6+ consecutive hours | Frozen readings |
| Dead dust channel | PM2.5, PM10 or TSP: daily median below 2 µg/m³ on 10 or more days within 30 days | Channels that failed and keep reporting about 1 µg/m³ |
| Cluster outlier | Station daily average > 3 robust standard deviations from its cluster median | A station that disagrees with its neighbourhood |
| Duplicate sources | Same station and hour from two sources | Double counting |

**`rest_of_kz`** (most towns have a single monitor, so a station is compared only with itself; applied from 10 Oct 2026)

| Rule | Check | What it catches |
|:-----|:------|:----------------|
| Range | Negative values, or ≥ 100 mg/m³ | Corrupt readings |
| Hard cap | Same limits as the city files | Implausible readings |
| Instrument ceiling | PM10 exactly 1,000 µg/m³ | Saturated analyser |
| Flatline | Identical value for 24+ consecutive hours | Frozen or dead channel |
| Zero flatline | Particulates (PM2.5, PM10, PMtot) at zero for 6+ consecutive hours | Dead channel |
| Dead dust channel | PM2.5, PM10 or PMtot: daily median below 2 µg/m³ on 10 or more days within 30 days | Channels that failed and keep reporting about 1 µg/m³ |

Shorter runs of identical values are kept in `rest_of_kz`: most are readings at the analyser's detection limit (e.g. H₂S 0.001 mg/m³), i.e. real "below detection" hours. One-hour peaks are kept as well — near industry they are real plumes, and a single station cannot tell an event from a glitch. Earlier versions of this page listed statistical-outlier and spike-detection stages; they were never applied to the published files.

Measurements flagged as suspect or invalid are **excluded** from these files. The full methodology is documented at [airdata.kz/methodology](https://airdata.kz/en/methodology/).

---

## Unit Conversions

| Source | Original | Published | Conversion |
|:-------|:---------|:----------|:-----------|
| KGMT (concentrations) | mg/m³ | µg/m³ | × 1,000 |
| KGMT (pressure) | mmHg | hPa | × 1.33322 |
| WAQI (PM2.5, PM10) | AQI index | µg/m³ | EPA breakpoint reverse |
| All others | µg/m³ | µg/m³ | No conversion |

Original values are preserved in the `raw_value` and `raw_unit` columns for traceability.

---

## Quick Start

### Python

```python
import pandas as pd

# Read Almaty PM2.5 hourly data
df = pd.read_csv('almaty/pm25.csv.gz')

# Read daily summaries
daily = pd.read_csv('almaty/daily/pm25.csv.gz', parse_dates=['date'])

# Filter to 2024
df_2024 = df[df['datetime_utc'].str.startswith('2024')]
```

### R

```r
library(readr)
df <- read_csv("almaty/pm25.csv.gz")
daily <- read_csv("almaty/daily/pm25.csv.gz")
```

### Command line

```bash
# Preview data
gzcat almaty/pm25.csv.gz | head -5

# Count rows
gzcat almaty/pm25.csv.gz | wc -l

# Extract to uncompressed CSV
gzcat almaty/pm25.csv.gz > almaty_pm25.csv
```

---

## Coverage

| City | PM2.5 since | Parameters | Hourly rows | Daily rows |
|:-----|:------------|:-----------|:------------|:-----------|
| Almaty | March 2017 | 9 | 2.9M | 15K |
| Astana | January 2018 | 11 | 1.9M | 19K |
| Karaganda | January 2018 | 12 | 1.2M | 23K |
| Rest of KZ | June 2020 | 9 | 22.6M | 402K |

---

## Known Limitations

- **Almaty 2017–2020**: PM2.5 data comes primarily from AirKaz low-cost sensors (daily granularity, lower accuracy than government reference monitors). Multi-parameter hourly data starts in 2020.
- **Karaganda 2019**: No data available — gap in source coverage.
- **Astana 2019**: Limited to PM2.5 only (other parameters start 2020).
- **rest_of_kz**: Uses `pm2_5` and `pmtot` codes instead of `pm25` and `tsp` (matches KGMT national naming convention).
- **Station coordinates**: Some historical stations lack lat/lon coordinates (shown as empty in CSV).
- **rest_of_kz stations without a city**: KazHydroMet's API gives no station metadata, and our station list covers 97 of the 323 stations in `rest_of_kz`. The others appear in the hourly files with an empty `city`, `lat` and `lon`, and are not part of the daily files (a daily value is a per-city average).
- **KazHydroMet gap (22 Dec 2025 – 17 Mar 2026)**: KazHydroMet data was not collected in this period; it cannot be backfilled because the KazHydroMet API serves only the latest hour.
- **Almaty OpenAQ**: since March 2026 the only OpenAQ provider left in Almaty is AirGradient, whose sensors are taken directly from AirGradient (source `airgradient`), so the `openaq` source has no Almaty rows after 18 Mar 2026.
- **US Embassy (WAQI)**: the US Embassy feed in Almaty has reported no PM2.5 since December 2025.
- **KazHydroMet dust channels**: many KazHydroMet PM2.5, PM10 and TSP channels have failed over the years and report about 1 µg/m³; those periods are removed (see Data Quality and the correction of 10 Oct 2026), so fewer government monitors contribute in 2022–2025.
- **PM2.5 equal to PM10 at some KazHydroMet stations**: at several stations (e.g. Karaganda PCP #8, Temirtau PCP #2) the PM2.5 channel repeats the PM10 value hour after hour, which is not a separate PM2.5 measurement and inflates PM2.5 there, notably in the `karaganda` files; under review.
- **Collection gap (25 Apr – 8 Oct 2026)**: the pipeline was offline. Almaty, Astana and Karaganda have no data for this period; `rest_of_kz` has data up to 21 Aug 2026. Collection resumed on 8 Oct 2026; the gap is not backfilled.

---

## Data Corrections

### 10 Oct 2026 — Dead KazHydroMet dust channels removed (PM2.5, PM10, TSP)

KazHydroMet dust channels fail by dropping to about 1 µg/m³ (with small jitter) or to zero and stay there for months. The earlier
rules did not catch this because the values are not identical, so these readings were published and pulled the city averages down.
A new rule (see "Data Quality") removes a station-day whose median is below 2 µg/m³ when the station has ten or more such days within
30 days. Please re-download all `pm25`, `pm10`, `tsp`, `pm2_5` and `pmtot` files.

- **Almaty:** 10 of the 11 KazHydroMet PM2.5 stations had dead periods (135,000 hourly values removed; PM10 107,000; TSP 86,000). 28,000 values of
  healthy stations, which had been rejected as outliers against their dead neighbours, are published again.
- **Annual mean of the daily city PM2.5 averages, before → after (µg/m³):**

| | 2021 | 2022 | 2023 | 2024 | 2025 |
|:--|:--|:--|:--|:--|:--|
| Almaty | 35.8 → 36.8 | 26.7 → 35.0 | 20.7 → 29.5 | 15.8 → 22.8 | 15.0 → 26.0 |
| Astana | 30.4 → 35.7 | 54.8 → 66.4 | 26.2 → 36.7 | 35.8 → 55.5 | 13.1 → 19.2 |
| Karaganda | 63.5 → 74.2 | 77.0 → 89.3 | 67.0 → 76.1 | 81.1 → 94.7 | 112.1 → 137.2 |

  **If you used the earlier files, the fall of Almaty's PM2.5 between 2021 and 2025 was mostly an artefact of failing monitors.**
- **Astana:** 40,000 PM2.5 and 16,000 PM10 values removed (4 stations). **Karaganda:** 20,000 PM2.5, 3,000 PM10 and 34,000 TSP values (one station).
- **`rest_of_kz`:** 1.26 million further values removed (`pm2_5` 706,000, `pm10` 317,000, `pmtot` 239,000).
- The rule was validated against Almaty's independent sensors: among them it removes only five broken sensors and the dead US Embassy feed, and it
  catches 93.5% of the KazHydroMet station-days that the independent network contradicts.

### 10 Oct 2026 — Quality control for `rest_of_kz`

Until now `rest_of_kz` was published with a range filter only. The rules in "Data Quality" above now apply; please re-download the `rest_of_kz` files.

- **3.43 million of 27.3 million hourly values removed (12.6%)**: 112,000 at or above a hard cap (PM2.5 up to 99,790 µg/m³), 46,000 PM10 values at the analyser's ceiling of exactly 1,000 µg/m³, 2.36 million in flatlines of 24 hours or longer, and 910,000 zero-dust hours of dead channels (nearly half of `pmtot`). No remaining value was changed.
- **Effect on averages** (all published values): PM2.5 44.5 → 23.8 µg/m³, PM10 52.7 → 27.9, O₃ 370 → 31.5 (thousands of placeholder values of 20,000), SO₂ 46.9 → 35.0.
- **Daily files are per city only.** Stations whose city we do not know (226 of 323) were averaged into one line per day with an empty city name — a mean over dozens of unrelated towns. Those lines are gone (about 12,600); the stations remain in the hourly files. About 49,600 city-days were dropped because no clean reading was left.
- Two city names lost a trailing space (`Semey`, `Kenkiyak vil.`).

### 10 Oct 2026 — Karaganda city stations, Almaty WAQI, KazHydroMet revisions and cleanup

All files were regenerated; please re-download the `almaty`, `karaganda` and `rest_of_kz` files if you use earlier copies.

- **Karaganda city monitors are now in the `karaganda` files.** Three KazHydroMet monitors inside Karaganda city (stations 36, 64, 65 — spelled "Karagandy" in the source) had been filed under `rest_of_kz` since 2021; about 654,000 hourly values moved. The `karaganda` files cover the KazHydroMet monitors of Karaganda city, Temirtau, Saran, Balkhash and Zhezkazgan; the city daily value averages these stations.
- **Almaty WAQI.** Six independent WAQI stations in Almaty were collected but never used because the routing relied on station names; they are now in the `almaty` files from 17 Mar 2026 (about 12,000 hourly values). WAQI stations that re-publish KazHydroMet monitors are excluded to avoid double counting.
- **One row per station and hour for WAQI.** Older WAQI data in the `almaty` files had several readings per station-hour; each station-hour is now one row with the average (244,000 rows → 70,000).
- **AirGradient hourly values.** The newest hour of each nightly run was stored as a partial average and never completed; 4,536 such hours were recomputed from the full hour, and new data is completed automatically.
- **KazHydroMet revisions.** KazHydroMet revises hourly values shortly after publishing them (in our measurement about 45% of values changed, by a median of 3%, within 20 minutes). From 9 Oct 2026 the revised values are stored; earlier hours keep the first published value.
- **Values exactly at a hard cap are invalid** (instrument ceilings or placeholders, e.g. 2000 µg/m³ SO₂).
- **Removed non-KazHydroMet rows:** 49,793 sub-hourly points of a unit-mislabelled stream from the Astana KazHydroMet data, and about 820,000 WAQI-derived and mislabelled rows from `rest_of_kz` (mostly `pm2_5`).

### 9 Oct 2026 — Hourly timestamps of KazHydroMet data corrected

Hourly KazHydroMet values (the `kgmt` source in the `almaty`, `astana` and `karaganda` files, and all of `rest_of_kz`) now
use the same convention as every other source: `datetime_utc` is the **start** of the measured hour, in UTC.

- Values loaded from the historical archive (2020-06 → 2025-12) were stamped **6 hours late**; they were moved back by 6 hours.
- KazHydroMet stamps every value with the **end** of its hour; all KazHydroMet hours were additionally moved back by 1 hour.
  In total, history before 2023-10-29 moved back by 7 hours and later data by 1 hour.
- Duplicate measurements created by the old error were removed (about 27,000 hourly values in the city files; about 670,000
  raw readings behind `rest_of_kz`).
- With the duplicates gone, several stuck or constant KazHydroMet sensors (e.g. TSP readings of 0.00 µg/m³) became visible;
  quality control now excludes those hours, so some hourly files have fewer rows.
- Daily values change little (median 0.6–3.7% by city and pollutant); hourly profiles such as the daily NO2 cycle are now aligned
  with the other sources.
- `raw_value` is now written with at most 6 decimals (values in `value_ugm3` are unchanged by this).

If you downloaded these files before this date, please download them again.

### 8 Oct 2026

All files were regenerated with these fixes; please re-download if you use earlier copies.

- **Daily values use Kazakhstan calendar days.** Daily files and summaries previously
  grouped hours by America/Los_Angeles days (12–13 h off). They now use local days
  (Asia/Almaty). Almaty PM2.5 daily values changed by a median of 17%.
- **Hourly timestamps are written in UTC** (`+00:00`). The instants are unchanged; only the
  offset notation differs from earlier files (`-07:00` / `-08:00`).
- **WAQI hourly readings moved to their true hour.** 75,026 WAQI rows (Almaty, 2023–2026)
  had been stored 12–14 h late; corrected. 3,255 resulting duplicates were dropped.
- **No double counting of AirGradient sensors.** OpenAQ re-publishes AirGradient sensors;
  in Almaty they are now taken only from the direct AirGradient feed.
- **Under review:** several KazHydroMet PM2.5 monitors in Almaty disagree strongly with the
  dense AirGradient network. (The KazHydroMet timestamp question noted here was resolved on 9 Oct 2026, see above.)

---

## Citation

```
AirData.kz. Open Air Quality Dataset for Kazakhstan.
Global Shapers Almaty Hub, 2019–present.
https://airdata.kz
```

---

## License

**CC BY-NC 4.0** — free to use, share, and adapt for non-commercial purposes with attribution.

- You **may**: use for research, journalism, education, personal projects, non-profit work
- You **may not**: sell the data, include it in commercial products, or use it in paid services
- You **must**: credit AirData.kz and link to this repository

Full license: [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)

Upstream source terms also apply (KazHydroMet, OpenAQ, WAQI).

---

## About

AirData.kz is a non-profit open data project by [Global Shapers Almaty Hub](https://www.globalshapers.org/hubs/almaty-hub), an initiative of the World Economic Forum. We collect air pollution data from every source we can find, clean it carefully, and share it with everyone — for free, forever.
