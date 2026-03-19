# Datasheet for Dataset: AirData.kz Open Air Quality Data

Following the "Datasheets for Datasets" framework by Gebru et al. (2021), *Communications of the ACM*, 64(12), 86–92.

---

## Motivation

**For what purpose was the dataset created?**

AirData.kz was created to fill a critical gap in publicly available air quality data for Kazakhstan. Government monitoring data was scattered across agencies, reported in inconsistent formats and units, and often inaccessible to researchers and the public. Historical readings were routinely deleted after short retention periods. This dataset collects, harmonizes, cleans, and permanently archives air quality measurements from every available source — freely accessible for research, journalism, public health analysis, and civic awareness.

**Who created the dataset?**

The dataset was created and is maintained by volunteers from the Global Shapers Almaty Hub, an initiative of the World Economic Forum. AirData.kz operates as a non-profit, non-commercial open data project.

**Who funded the creation?**

The project is entirely self-funded by its volunteers. Infrastructure costs are covered through personal contributions and occasional public donations. There are no grants, corporate sponsors, or government funding.

---

## Composition

**What do the instances represent?**

Each instance is a single air quality or meteorological measurement: one parameter, at one station, at one point in time. For example: "PM2.5 = 45.2 µg/m³ at station kgmt_107 on 2024-01-15 at 14:00 UTC+6."

**How many instances are there?**

As of March 2026, the dataset contains approximately:
- 2.9 million cleaned hourly readings for Almaty (9 parameters)
- 1.9 million for Astana (11 parameters)
- 1.2 million for Karaganda (12 parameters)
- 28 million for the rest of Kazakhstan (9 parameters)
- 1,000+ monitoring stations in the registry

The dataset grows daily as new data is ingested every 20 minutes from active sources.

**Does the dataset contain all possible instances or is it a sample?**

The dataset aims to be a census, not a sample — we collect every available reading from every source. However, it is inherently incomplete: government stations have downtime, sensors go offline, and some historical data was lost before archiving began in 2019. Coverage varies by city and time period. Almaty has the densest coverage (6 sources, 200+ stations). Other cities rely primarily on KGMT government stations.

**What data does each instance consist of?**

Each hourly measurement record contains:
- Station identifier and geographic coordinates
- Timestamp (UTC with timezone offset)
- Parameter code (e.g., pm25, no2, co)
- Measured value in harmonized units (µg/m³ for concentrations)
- Original raw value and unit (preserved for auditability)
- Data source identifier
- Geographic cluster assignment

Daily summary records contain city-wide averages computed from cluster averages, with statistics (median, min/max cluster, standard deviation, station count).

**Is there a label or target associated with each instance?**

No. This is an observational dataset, not a labeled dataset for supervised learning. Each measurement has passed a 7-stage quality control pipeline, but there is no target variable.

**Is any information missing?**

Yes. Common sources of missing data include: sensor downtime, network outages, government stations reporting only during business hours (early data), and parameters not measured by all stations. Missing values are never imputed or interpolated.

**Are there errors, sources of noise, or redundancies?**

Yes, extensively documented. Known issues include sensor drift (especially low-cost sensors), stuck sensors, sudden spikes from electromagnetic interference, and government station unit changes. The 7-stage cleaning pipeline specifically targets these issues.

**Is the dataset self-contained?**

Yes. The published CSV files are fully self-contained and do not require external resources.

**Does the dataset contain confidential data?**

No. All data consists of environmental measurements from fixed monitoring stations in public locations. No personal data is collected.

---

## Collection Process

**How was the data acquired?**

Data is directly observed by physical instruments (hardware sensors and reference-grade monitors). It is acquired via automated API polling from five active sources:

| Source | Method | Stations | Since |
|:-------|:-------|:---------|:------|
| KazHydroMet (KGMT) | Government REST API | 141+ | 2018 |
| AirGradient | Public sensor API | 139 | 2026 |
| OpenAQ | Open data API (v3) | 22 | 2020 |
| WAQI / aqicn.org | Public feeds | 11 | 2023 |
| AirKaz | Historical CSV archives | 41 | 2017–2020 |

Additionally, historical government data (2018–2022) was extracted from Excel spreadsheets provided directly by KazHydroMet under an official data-sharing agreement.

**Who was involved in the data collection?**

Upstream data collection is performed by the operating organizations. AirData.kz's role is aggregation, harmonization, cleaning, and archival. Pipeline development and maintenance is done by project volunteers.

**Over what timeframe was the data collected?**

Earliest records: March 2017 (AirKaz PM2.5 sensors in Almaty). Collection is continuous and ongoing — new data is ingested every 20 minutes.

---

## Preprocessing, Cleaning, and Labeling

**Was any preprocessing/cleaning done?**

Yes, extensively. Every measurement passes a 7-stage cleaning pipeline:

| Stage | Check | Action |
|:------|:------|:-------|
| S1 | Negative values and nulls | Rejected at ingestion |
| S2 | Hard cap (physically impossible values) | Flagged invalid |
| S3 | Constant/dead sensor detection | Flagged suspect |
| S4 | Statistical outlier (robust Z-score > 10 with partial pooling) | Flagged invalid |
| S5 | Singleton spike (isolated >10x jump) | Flagged invalid |
| S6 | Stuck sensor (identical value for 6+ hours) | Flagged suspect |
| S7 | Cluster outlier (station daily avg >3 robust-Z from cluster) | Flagged invalid |

Only measurements that pass all stages are included in these files.

Unit harmonization is also performed: KGMT mg/m³ → µg/m³ (×1000), WAQI AQI index → µg/m³ (EPA breakpoint reverse conversion).

**Was the raw data saved?**

Yes. Raw data is preserved exactly as received in dedicated database tables. Every published measurement retains the original value and unit in the `raw_value` and `raw_unit` columns.

**Is the cleaning software available?**

Yes. The entire pipeline is open source: [github.com/qazybekb/airdata](https://github.com/qazybekb/airdata)

---

## Uses

**Has the dataset been used for any tasks?**

Yes:
- AirData-AI — AI-powered natural language analytics tool at [airdata.kz/ask](https://airdata.kz/en/ask/)
- Calendar heatmap visualizations showing daily PM2.5 levels since 2017
- Cigarette equivalent calculator based on Berkeley Earth methodology
- Personal exposure estimator based on daily activity patterns

**What other tasks could the dataset be used for?**

Epidemiological research, urban planning analysis, climate studies, machine learning research (time-series anomaly detection, air quality forecasting), environmental journalism, and education.

**Are there tasks for which the dataset should not be used?**

The dataset should not be used for: real-time health emergency alerts (use official government sources), regulatory compliance or legal proceedings, or individual health risk assessment without professional guidance.

---

## Distribution

**How is the dataset distributed?**

Gzip-compressed CSV files organized by city and parameter. Available on GitHub ([AirDatakz-OpenData](https://github.com/qazybekb/AirDatakz-OpenData)) and at [airdata.kz](https://airdata.kz).

**License?**

CC BY-NC 4.0 — free for non-commercial use with attribution. Commercial use is prohibited.

---

## Maintenance

**Who is maintaining the dataset?**

The AirData.kz volunteer team, operating under the Global Shapers Almaty Hub.

**How can the maintainer be contacted?**

Email: airdatakz@gmail.com

**Will the dataset be updated?**

Yes, daily. New data is ingested every 20 minutes, cleaned overnight, and published each morning. The GitHub repository is updated automatically after each export.

**Will older versions continue to be available?**

Historical data is never deleted — the dataset is append-only. Git history preserves previous versions. Quality flags may be updated as the cleaning methodology improves, but raw values are immutable.

**Can others contribute?**

Yes. The project is open source. Contributors can submit issues or pull requests via GitHub, suggest new data sources, or report data quality issues.

---

## Citation

```
AirData.kz. Open Air Quality Dataset for Kazakhstan.
Global Shapers Almaty Hub, 2019–present.
https://github.com/qazybekb/AirDatakz-OpenData
```
