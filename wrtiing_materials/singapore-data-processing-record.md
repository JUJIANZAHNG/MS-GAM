# Singapore

## 1. Tourism data

| Item | Value |
|---|---|
| Raw source institution | Singapore Tourism Board (STB) via SingStat Table Builder |
| Raw source page / API | `https://tablebuilder.singstat.gov.sg/api/table/tabledata/M550001` |
| Raw carrier | JSON (Table Builder API, series M550001) |
| Raw frequency | monthly |
| Current official processed file | `data_processed/tourism/singapore_arrivals_month.csv` |
| Time range | 2008-01 to 2026-04 |
| Note | official SingStat/STB total international visitor arrivals series |

## 2. Climate data

Singapore has two parallel climate layers that are NOT bridged in the processed data. Distinguish them carefully.

### 2.1 Layer 1 — Monthly (NEA / data.gov.sg, primary)

| Item | Value |
|---|---|
| Raw source institution | Singapore NEA (National Environment Agency) via data.gov.sg |
| Raw source page / API | `https://data.gov.sg/api/action/datastore_search` |
| Raw carrier | JSON (data.gov.sg `datastore_search`; 4 separate monthly series) |
| Series fetched | `mean_temperature_c`, `mean_daily_max_temperature_c`, `mean_daily_min_temperature_c`, `total_rainfall_mm` (4 datasets) |
| Raw frequency | monthly |
| Current official processed file | `data_processed/climate/singapore_monthly_weather_summary.csv` |
| Row count | 533 |
| Time range | 1982-01 to 2026-05 |
| Source summary | `data_processed/climate/singapore_climate_source_summary.csv` (4 series, resource_id, start_month, end_month) |

### 2.2 Layer 2 — Daily (Changi S24, parallel, NOT in monthly ready panel)

| Item | Value |
|---|---|
| Raw source institution | Singapore Meteorological Services (MSS) via weather.gov.sg |
| Raw source page / API | `https://www.weather.gov.sg/climate-historical-daily/` |
| Raw carrier | HTML (parsed from weather.gov.sg climate-historical-daily page; Changi Meteorological Station, station code S24) |
| Raw frequency | daily |
| Current official processed file | `data_processed/climate/singapore_changi_daily_weather.csv` |
| Row count | 6,726 |
| Time range | 2008-01-01 to 2026-05-31 |
| Also | `data_processed/climate/singapore_changi_daily_weather_fetch_log.csv` (per-month raw HTML fetch manifest) |
| Variables | `daily_rainfall_total_mm`, `highest_30min_rainfall_mm`, `highest_60min_rainfall_mm`, `highest_120min_rainfall_mm`, `mean_temperature_c`, `maximum_temperature_c`, `minimum_temperature_c`, `mean_wind_speed_kmh`, `max_wind_speed_kmh` |

**KEY POINT.** Monthly tourism is monthly; monthly climate is monthly (NEA / data.gov.sg); Changi daily is daily. The two frequencies are NOT bridged. The monthly ready panel uses the NEA monthly series, NOT the Changi daily series aggregated to monthly.

## 3. Extreme-weather data

The current material does NOT have a dedicated extreme-weather processed layer for Singapore. Heavy-rain day counts, hot day counts, and cold day counts are not extracted in the current processed layer. Changi daily fields include `daily_rainfall_total_mm` and the `highest_30/60/120min_rainfall_mm` columns, but no monthly extreme-weather index is derived from them. The 4-series NEA monthly panel contains only `mean_temperature_c`, `mean_daily_max_temperature_c`, `mean_daily_min_temperature_c`, and `total_rainfall_mm` — no heavy-rain / hot / cold day counts.

## 4. Processing steps

### 4.1 Monthly tourism (input → intermediate → output)

- **Input**: SingStat Table Builder API `M550001` JSON for total international visitor arrivals.
- **Intermediate processing**: JSON flatten to a single monthly series keyed by `month`; standardize `arrivals` column.
- **Output**: `data_processed/tourism/singapore_arrivals_month.csv` (2008-01 to 2026-04).
- **Script**: `scripts/data_fetch/fetch_singapore_arrivals.py`.

### 4.2 Monthly climate — NEA / data.gov.sg (input → intermediate → output)

- **Input**: 4 data.gov.sg `datastore_search` JSON series (`mean_temperature_c`, `mean_daily_max_temperature_c`, `mean_daily_min_temperature_c`, `total_rainfall_mm`).
- **Intermediate processing**: parse each series independently; merge the 4 series by `month` into a single monthly panel.
- **Output**: `data_processed/climate/singapore_monthly_weather_summary.csv` (533 rows, 1982-01 to 2026-05).
- **Also**: `data_processed/climate/singapore_climate_source_summary.csv` records the 4 series, their `resource_id`, and per-series `start_month` / `end_month` / `row_count`.
- **Script**: `scripts/data_fetch/fetch_singapore_climate.py`.

### 4.3 Daily climate — Changi S24 (input → intermediate → output)

- **Input**: HTML from `https://www.weather.gov.sg/climate-historical-daily/` (per month, per station).
- **Intermediate processing**: scrape the climate-historical-daily page; submit the records form for Changi (S24); parse the HTML to per-day `daily_rainfall_total_mm`, `highest_30min_rainfall_mm`, `highest_60min_rainfall_mm`, `highest_120min_rainfall_mm`, `mean_temperature_c`, `maximum_temperature_c`, `minimum_temperature_c`, `mean_wind_speed_kmh`, `max_wind_speed_kmh`; log the raw HTML path per month.
- **Output**: `data_processed/climate/singapore_changi_daily_weather.csv` (6,726 rows, 2008-01-01 to 2026-05-31).
- **Also**: `data_processed/climate/singapore_changi_daily_weather_fetch_log.csv` (per-month raw HTML fetch manifest, with `rows` per month, `raw_html_path`, and overall `start_date` / `end_date`).
- **Script**: `scripts/data_fetch/fetch_singapore_climate_daily.py`.

### 4.4 Monthly ready panel (input → intermediate → output)

- **Input**: `data_processed/tourism/singapore_arrivals_month.csv` (monthly tourism) + `data_processed/climate/singapore_monthly_weather_summary.csv` (NEA monthly climate).
- **Intermediate processing**: left-join by `month`; carry over `arrivals`, `mean_temperature_c`, `mean_daily_max_temperature_c`, `mean_daily_min_temperature_c`, `total_rainfall_mm`; record `tourism_source_file`, `climate_source_file`, `has_complete_climate`.
- **Output**: `data_processed/climate/singapore_tourism_climate_ready_panel.csv` (220 rows, 2008-01 to 2026-04).
- **Also**: `data_processed/climate/singapore_tourism_climate_ready_summary.csv` (1-row summary: `tourism_start_month` = 2008-01, `tourism_end_month` = 2026-04, `row_count` = 220, `missing_climate_rows` = 0).

## 5. Missingness / gap filling / aggregation

| Layer | Gap fill | Aggregation | Note |
|---|---|---|---|
| Monthly tourism (SingStat M550001) | no fill | none (already monthly) | series used as-is |
| Monthly climate (NEA / data.gov.sg) | no fill | none (already monthly) | 4 series merged by month |
| Daily climate (Changi S24) | no fill | none (kept at daily resolution) | raw HTML manifest in fetch log |
| Monthly ready panel | no fill | NONE at file level | both inputs are already monthly; Changi daily is NOT aggregated into the ready panel |

**KEY POINT.** The Changi daily series is NOT aggregated into the monthly ready panel. The ready panel uses the NEA monthly series, not the Changi daily series aggregated to monthly. The two climate frequencies are not bridged.

## 6. Final processed files

| file | role | frequency | start | end | note |
|---|---|---|---|---|---|
| `data_processed/tourism/singapore_arrivals_month.csv` | monthly tourism (SingStat M550001) | monthly | 2008-01 | 2026-04 | primary Singapore monthly arrivals series |
| `data_processed/climate/singapore_monthly_weather_summary.csv` | monthly climate (NEA / data.gov.sg) | monthly | 1982-01 | 2026-05 | 533 rows; 4-series merge by month |
| `data_processed/climate/singapore_changi_daily_weather.csv` | daily climate (Changi S24) | daily | 2008-01-01 | 2026-05-31 | 6,726 rows; daily layer; parallel to monthly climate; not in monthly ready panel |
| `data_processed/climate/singapore_changi_daily_weather_fetch_log.csv` | daily climate fetch manifest | per-month log | 2008-01 | 2026-05 | raw HTML path and row count per month |
| `data_processed/climate/singapore_climate_source_summary.csv` | monthly climate source index | 4-series index | 1982-01 | 2026-05 | `resource_id` / `dataset_page` / `row_count` per series |
| `data_processed/climate/singapore_tourism_climate_ready_panel.csv` | monthly ready panel (tourism × NEA monthly climate) | monthly | 2008-01 | 2026-04 | 220 rows; Changi daily is parallel and not in this panel |
| `data_processed/climate/singapore_tourism_climate_ready_summary.csv` | monthly ready panel summary | 1-row summary | 2008-01 | 2026-04 | row_count=220, missing_climate_rows=0 |

## 7. Paper-ready description

**English.** Singapore monthly visitor arrivals are taken from the SingStat Table Builder official series (M550001) for total international visitor arrivals, covering 2008-01 to 2026-04. Singapore normal-climate variables — monthly mean temperature, mean daily maximum / minimum temperature, and monthly total rainfall — are from the data.gov.sg official monthly climate series issued by the National Environment Agency (NEA), merged by month across four separate `datastore_search` series and covering 1982-01 to 2026-05. A parallel daily Changi Meteorological Station (S24) climate series is from weather.gov.sg's `climate-historical-daily` records, covering 2008-01-01 to 2026-05-31. The two climate frequencies are NOT bridged: the monthly ready panel uses the NEA monthly series, not the Changi daily series aggregated to monthly. The ready panel is the left-join of monthly tourism and NEA monthly climate keyed by month. The current materials do not have a dedicated extreme-weather processed layer for Singapore; heavy-rain / hot / cold day counts are not extracted.

**中文。** 新加坡月度访客数取自 SingStat Table Builder 官方系列 M550001（国际访客总量），覆盖 2008-01 至 2026-04。新加坡常态气候变量（月平均气温、月平均日最高/最低气温、月总降水量）取自 data.gov.sg 由新加坡国家环境局（NEA）发布的官方月度气候系列，由 4 条独立的 `datastore_search` 系列按月合并得到，覆盖 1982-01 至 2026-05。另有樟宜气象站（S24）日度气候序列取自 weather.gov.sg 的 `climate-historical-daily` 日度记录页，覆盖 2008-01-01 至 2026-05-31。两套气候频率并未桥接：月度 ready 面板用的是 NEA 月度系列，不是把樟宜日度按月聚合。Ready 面板由月度访客与 NEA 月度气候按月主键左连接得到。当前材料中尚无专门的新加坡极端天气处理层；暴雨日、高温日、低温日计数尚未抽取。
