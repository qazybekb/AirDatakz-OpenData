# Data Dictionary

Column-level documentation for all CSV files in this repository.

---

## Hourly Files — `{city}/{parameter}.csv.gz`

| # | Column | Type | Nullable | Description |
|:--|:-------|:-----|:---------|:------------|
| 1 | `datetime_utc` | timestamp | no | Measurement time in UTC (with timezone offset) |
| 2 | `station_id` | string | no | Unique station identifier (e.g., `kgmt_107`, `waqi_12774`) |
| 3 | `station_name` | string | yes | Human-readable station name / address |
| 4 | `source` | string | no | Data source: `kgmt`, `openaq`, `waqi`, `airgradient`, `airkaz` |
| 5 | `lat` | float | yes | Station latitude (WGS84). Null for some historical stations. |
| 6 | `lon` | float | yes | Station longitude (WGS84) |
| 7 | `value_ugm3` | float | no | Measured value in harmonized units (see Parameters below) |
| 8 | `raw_value` | float | yes | Original value before unit conversion |
| 9 | `raw_unit` | string | yes | Original unit from the source (e.g., `mg/m3`, `µg/m3`) |
| 10 | `conversion_note` | string | yes | Conversion applied: `mg_to_ug`, `aqi_to_ugm3`, `exact` (no conversion) |
| 11 | `cluster_id` | integer | yes | Geographic cluster number (0-9 for Almaty, 0-4 for Astana) |
| 12 | `cluster_name` | string | yes | Cluster name (e.g., `City Center`, `North Central`) |

## Daily Files — `{city}/daily/{parameter}.csv.gz`

| # | Column | Type | Nullable | Description |
|:--|:-------|:-----|:---------|:------------|
| 1 | `date` | date | no | Calendar date (YYYY-MM-DD) |
| 2 | `value_ugm3` | float | no | City-wide daily average (mean of cluster averages) |
| 3 | `median_value` | float | no | Median of cluster daily averages |
| 4 | `min_cluster` | float | yes | Lowest cluster daily average |
| 5 | `max_cluster` | float | yes | Highest cluster daily average |
| 6 | `std_cluster` | float | yes | Standard deviation across cluster averages |
| 7 | `n_clusters` | integer | no | Number of geographic clusters reporting that day |
| 8 | `n_stations` | integer | no | Total monitoring stations reporting that day |

## Rest of KZ — `rest_of_kz/{parameter}.csv.gz`

Same as hourly files above, but with an additional `city` column (column 4) identifying the city. No `cluster_id` or `cluster_name` columns.

| # | Column | Type | Description |
|:--|:-------|:-----|:------------|
| 1 | `datetime_utc` | timestamp | Measurement time |
| 2 | `station_id` | string | KGMT station ID |
| 3 | `station_name` | string | Station name |
| 4 | `city` | string | City name (Russian) |
| 5 | `lat` | float | Latitude |
| 6 | `lon` | float | Longitude |
| 7 | `value_ugm3` | float | Value in µg/m³ |
| 8 | `raw_value` | float | Original value |
| 9 | `raw_unit` | string | Original unit |

---

## Parameters

| Code | Full Name | Unit | WHO 24h Guideline |
|:-----|:----------|:-----|:------------------|
| `pm25` / `pm2_5` | PM2.5 (fine particulate matter, <2.5µm) | µg/m³ | 15 µg/m³ |
| `pm10` | PM10 (coarse particulate matter, <10µm) | µg/m³ | 45 µg/m³ |
| `co` | Carbon monoxide | µg/m³ | — |
| `no2` | Nitrogen dioxide | µg/m³ | 25 µg/m³ |
| `no` | Nitric oxide | µg/m³ | — |
| `so2` | Sulfur dioxide | µg/m³ | 40 µg/m³ |
| `o3` | Ozone | µg/m³ | 100 µg/m³ |
| `h2s` | Hydrogen sulfide | µg/m³ | — |
| `tsp` / `pmtot` | Total suspended particles | µg/m³ | — |
| `nh3` | Ammonia | µg/m³ | — |
| `ch4` | Methane | µg/m³ | — |
| `thc` | Total hydrocarbons | µg/m³ | — |
| `co2` | Carbon dioxide | ppm | — |

---

## Geographic Clusters

Stations are grouped into geographic clusters using K-Means on coordinates. Daily city averages are computed as the mean of cluster averages (not the mean of all stations), preventing dense areas from dominating.

### Almaty (10 clusters)

| ID | Name | Description |
|:---|:-----|:------------|
| 0 | South Foothills | Medeu, Kok-Tobe — mountain foothills, cleanest air |
| 1 | Southwest Suburbs | Kyrgauldy, Abai village — low-density residential |
| 2 | South Central | KazGU area — university district |
| 3 | West Residential | US Embassy, Ryskulova — western residential |
| 4 | City Center | Tole Bi, Satpayeva, Gorky Park — downtown core |
| 5 | West Central | Almaty Arena, Denderpark — mixed use |
| 6 | East Suburbs | Talgar, Tuzdybastau — satellite towns |
| 7 | North Central | Turksib, Zhetysu — highest PM2.5 |
| 8 | Far West | Shamalghan — rural outskirts |
| 9 | Far North | Zhana Kuat, Otegen Batyr — northern outskirts |

### Astana (5 clusters)

| ID | Name |
|:---|:-----|
| 0–4 | Geographic zones across the city |

### Karaganda (1 cluster)

All stations in a single cluster (includes Temirtau, Balkhash, Zhezkazgan, Saran).
