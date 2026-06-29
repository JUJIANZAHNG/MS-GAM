# Hawaii

## 1. Tourism data

- raw source: UHERO / Hawaii DBEDT Tourism Data Warehouse
- raw carrier: JSON (UHERO TDW API `https://api.uhero.hawaii.edu/dvw/series/trend?i=VV101&m=MM102&d=DI10&f=M`)
  - indicator: `VV101` (Visitor arrivals)
  - market: `MM102` (Air total)
  - destination: `DI10` (Statewide)
  - frequency: `M` (monthly)
- raw frequency: monthly
- current official processed file: `data_processed/tourism/hawaii_arrivals_month.csv`
- time range: 1989-01 to 2026-04 (449 lines incl. header; 448 data rows; end row = 2026-04-01)

## 2. Climate data

Two parallel layers exist. They are different frequencies, different aggregation scopes, and different source APIs. They are NOT bridged.

### Layer 1 (monthly, primary; used in the ready panel)

- raw source: NOAA NCEI Climate at a Glance, statewide time series 51 (Hawaii)
- raw carrier: JSON (NCEI statewide monthly API for `tavg` and `pcp`)
  - tavg URL: `https://www.ncei.noaa.gov/access/monitoring/climate-at-a-glance/statewide/time-series/51/tavg/all?format=json`
  - pcp URL: `https://www.ncei.noaa.gov/access/monitoring/climate-at-a-glance/statewide/time-series/51/pcp/all?format=json`
- raw frequency: monthly (statewide)
- current official processed file: `data_processed/climate/hawaii_monthly_weather_summary.csv` (425 rows, 1991-01 to 2026-05)
- columns: `destination, month, source_institution, frequency, source_page, source_api, mean_temperature_f, mean_temperature_c, total_precipitation_in, total_precipitation_mm, source_note, cross_covid, processed_file`
- Unit conversion: Fahrenheit -> Celsius; inches -> millimeters (applied at processing time; both raw and converted columns are kept).

### Layer 2 (daily, parallel, multi-station pool; not in the ready panel)

- raw source: NOAA NCEI GHCN-Daily
- raw carrier: fixed-width `.dly` per station per element (TMAX / TMIN / TAVG / PRCP)
- raw frequency: daily
- default station pool (7 stations):
  - USW00022521 HONOLULU INTL AP
  - USW00021504 HILO INTL AP
  - USW00021510 KAILUA KONA KE-AHOLE AP
  - USW00022516 KAHULUI AP
  - USW00022536 LIHUE WSO AP
  - USC00519397 WAIKIKI
  - USC00511484 HILO 86A
- processed files (all in `data_processed/climate/`):
  - `hawaii_ghcn_daily_weather.csv` (76,205 rows; 1989 -> 2026) - default 7-station daily panel
  - `hawaii_ghcn_station_summary.csv` - station-level summary (default pool)
  - `hawaii_ghcn_station_inventory_screen.csv` - inventory screen for available stations
  - `hawaii_ghcn_common_stations_*.csv` - many relaxed coverage windows (see section 6)
  - `hawaii_ghcn_daily_weather_modern32_1998_2025.csv` - modern32 station pool, 1998-2025
  - `hawaii_ghcn_daily_weather_relaxed_1998_2023_window.csv` - relaxed pool, 1998-2023 window
  - `hawaii_ghcn_station_actual_coverage_all269_1998_2025.csv` - actual coverage of all-269 inventory
  - `hawaii_ghcn_station_actual_coverage_modern32_1998_2025.csv` - actual coverage of modern32 pool
- current official processed file for the daily layer: `hawaii_ghcn_daily_weather.csv` (76,205 rows)

Key point: statewide monthly is the NCEI Climate at a Glance monthly API; GHCN-Daily is a station-pool daily series. The monthly ready panel uses the NCEI statewide monthly series, NOT the GHCN daily series aggregated to monthly.

GHCN station pool has MULTIPLE coverage windows, not a single fixed window: default 7 stations (typically 1998+), modern32 1998-2025, relaxed 1989-2025, relaxed 1998-2023, relaxed 2000-2024, and additional `_common_stations_YYYY_2025.csv` rolling windows. If the daily layer is used, the specific window must be specified.

## 3. Extreme-weather data

Current materials do not have a dedicated extreme-weather processed layer for Hawaii. No Hawaii-specific extreme-weather file is present in `data_processed/climate/`, and the scripts under `scripts/data_fetch/` for Hawaii do not include an extreme-weather fetcher. No daily or monthly extreme-weather index is produced for Hawaii in the current materials.

## 4. Processing steps

### Monthly tourism
- input: UHERO TDW API `series/trend?i=VV101&m=MM102&d=DI10&f=M` (JSON, with dimensions/indicators/markets/destinations lookups)
- intermediate: `data_raw/hawaii_tourism/*.json` (raw JSON saved); Python `fetch_hawaii_monthly_arrivals.py` flattens the series handle
- output: `data_processed/tourism/hawaii_arrivals_month.csv` (monthly total visitor arrivals, statewide air-total)

### Monthly climate (NCEI)
- input: NCEI Climate at a Glance JSON for `tavg` and `pcp` on statewide time series 51 (Hawaii)
- intermediate: `data_raw/hawaii_climate/tavg_all.json` and `pcp_all.json` (raw JSON saved); Python `fetch_hawaii_climate.py` parses values, applies Fahrenheit -> Celsius and inches -> millimeters
- output: `data_processed/climate/hawaii_monthly_weather_summary.csv` (425 rows, 1991-01 to 2026-05)

### Daily climate (GHCN station pool)
- input: NOAA NCEI GHCN-Daily fixed-width `.dly` files, downloaded from `https://www.ncei.noaa.gov/pub/data/ghcn/daily/all` per station per element (TMAX / TMIN / TAVG / PRCP), with station metadata from `ghcnd-stations.txt` and `ghcnd-inventory.txt`
- intermediate: `data_raw/hawaii_ghcn/*.dly` (raw files saved); Python `fetch_hawaii_ghcn_daily.py` parses each `.dly` into per-station daily records, cleans GHCN sentinels (e.g. -9999) at read-in, derives `tavg_c` from `tmax_c`/`tmin_c` when no native TAVG exists
- output: `data_processed/climate/hawaii_ghcn_daily_weather.csv` (76,205 rows) and the multiple relaxed-window sister files listed in section 2/6

### Monthly ready panel
- input: `hawaii_arrivals_month.csv` (monthly tourism) + `hawaii_monthly_weather_summary.csv` (NCEI monthly climate)
- intermediate: Python merge keyed by `month`; the 1989-01 to 1990-12 tourism rows are kept with climate columns null (climate starts 1991-01)
- output: `data_processed/climate/hawaii_tourism_climate_ready_panel.csv` (448 rows, 1989-01 to 2026-04)

## 5. Missingness / gap filling / aggregation

- Monthly tourism: no fill. Raw UHERO series is used as-is.
- Monthly climate (NCEI): no fill. Raw NCEI statewide monthly series is used as-is. Unit conversion is applied (F->C, in->mm).
- Daily climate (GHCN): no fill. GHCN sentinels such as -9999 are cleaned at read-in. Missing days/stations are kept as missing.
- Ready panel: no fill. The merge keeps tourism rows for 1989-01 to 1990-12 with climate columns null; the strict climate-complete window is 1991-01 to 2026-04 (recorded in `hawaii_tourism_climate_ready_summary.csv`).
- Aggregations: NONE at the ready-panel level. Both inputs are already monthly. The GHCN daily layer is NOT aggregated into the monthly ready panel; the ready panel uses the NCEI statewide monthly series, not the GHCN daily series aggregated to monthly.

## 6. Final processed files

| file | role | frequency | start | end | note |
|---|---|---|---|---|---|
| `hawaii_arrivals_month.csv` | monthly tourism (statewide air-total) | monthly | 1989-01 | 2026-04 | UHERO / DBEDT TDW; 448 data rows; VV101 / MM102 / DI10 |
| `hawaii_monthly_weather_summary.csv` | monthly climate (statewide) | monthly | 1991-01 | 2026-05 | NOAA NCEI Climate at a Glance time series 51; 425 rows; F->C and in->mm applied |
| `hawaii_climate_source_summary.csv` | climate source audit (2 series) | metadata | - | - | documents the two NCEI monthly series (tavg, pcp) |
| `hawaii_ghcn_daily_weather.csv` | daily climate (default 7-station pool) | daily | 1989-01 | 2026-06 | 76,205 rows; TMAX / TMIN / TAVG / PRCP per station per day |
| `hawaii_ghcn_station_summary.csv` | station-level summary (default pool) | metadata | - | - | default 7 stations; per-station coverage and elements |
| `hawaii_ghcn_station_actual_coverage_all269_1998_2025.csv` | actual coverage audit (all-inventory 269 stations) | metadata | 1998 | 2025 | station-pool coverage diagnostic |
| `hawaii_ghcn_station_actual_coverage_modern32_1998_2025.csv` | actual coverage audit (modern32 pool) | metadata | 1998 | 2025 | modern32 station-pool coverage diagnostic |
| `hawaii_ghcn_station_inventory_screen.csv` | station inventory screen | metadata | - | - | pre-selection screen of candidate stations |
| `hawaii_ghcn_common_stations_1989_2025.csv` | common-stations window | metadata | 1989 | 2025 | one of multiple relaxed coverage windows |
| `hawaii_ghcn_common_stations_1991_2025.csv` | common-stations window | metadata | 1991 | 2025 | one of multiple relaxed coverage windows |
| `hawaii_ghcn_common_stations_1992_2025.csv` | common-stations window | metadata | 1992 | 2025 | one of multiple relaxed coverage windows |
| `hawaii_ghcn_common_stations_1993_2025.csv` | common-stations window | metadata | 1993 | 2025 | one of multiple relaxed coverage windows |
| `hawaii_ghcn_common_stations_1994_2025.csv` | common-stations window | metadata | 1994 | 2025 | one of multiple relaxed coverage windows |
| `hawaii_ghcn_common_stations_1995_2025.csv` | common-stations window | metadata | 1995 | 2025 | one of multiple relaxed coverage windows |
| `hawaii_ghcn_common_stations_1996_2025.csv` | common-stations window | metadata | 1996 | 2025 | one of multiple relaxed coverage windows |
| `hawaii_ghcn_common_stations_1997_2025.csv` | common-stations window | metadata | 1997 | 2025 | one of multiple relaxed coverage windows |
| `hawaii_ghcn_common_stations_1998_2025.csv` | common-stations window | metadata | 1998 | 2025 | one of multiple relaxed coverage windows |
| `hawaii_ghcn_common_stations_1999_2025.csv` | common-stations window | metadata | 1999 | 2025 | one of multiple relaxed coverage windows |
| `hawaii_ghcn_common_stations_2000_2025.csv` | common-stations window | metadata | 2000 | 2025 | one of multiple relaxed coverage windows |
| `hawaii_ghcn_common_stations_relaxed_1995_2020.csv` | common-stations window (relaxed) | metadata | 1995 | 2020 | relaxed coverage window |
| `hawaii_ghcn_common_stations_relaxed_1998_2023.csv` | common-stations window (relaxed) | metadata | 1998 | 2023 | relaxed coverage window |
| `hawaii_ghcn_common_stations_relaxed_2000_2024.csv` | common-stations window (relaxed) | metadata | 2000 | 2024 | relaxed coverage window |
| `hawaii_ghcn_common_station_count_by_start_year_end_2025.csv` | count-by-start-year summary | metadata | - | 2025 | per-start-year common-station counts |
| `hawaii_ghcn_common_station_pool_all_inventory.csv` | full inventory pool | metadata | - | - | union of candidate stations from the full GHCN inventory |
| `hawaii_ghcn_daily_weather_modern32_1998_2025.csv` | daily GHCN (modern32 pool) | daily | 1998 | 2025 | daily GHCN station pool; multiple coverage windows exist |
| `hawaii_ghcn_daily_weather_relaxed_1998_2023_window.csv` | daily GHCN (relaxed pool) | daily | 1998 | 2023 | daily GHCN station pool; multiple coverage windows exist |
| `hawaii_ghcn_station_summary_relaxed_1998_2023_window.csv` | station summary (relaxed 1998-2023) | metadata | 1998 | 2023 | paired with the relaxed 1998-2023 daily file |
| `hawaii_tourism_climate_ready_panel.csv` | monthly ready panel (tourism x NCEI statewide monthly climate; GHCN daily is parallel and not in this panel) | monthly | 1989-01 | 2026-04 | 448 rows; merge key = month; climate columns null for 1989-01..1990-12 |
| `hawaii_tourism_climate_ready_summary.csv` | ready-panel summary | metadata | 1989-01 | 2026-04 | 1 row; records `tourism_start_month`, `tourism_end_month`, `climate_complete_start_month` (1991-01), `climate_complete_end_month` (2026-04), `missing_climate_rows=24` |

Note on `hawaii_ghcn_common_stations_*.csv`: many window files coexist (annual-start 1989-2020 with end 2025; relaxed 1995-2020, 1998-2023, 2000-2024). The `*_common_stations_*.csv` family is a multi-window set; if a specific window is referenced, name it explicitly.

## 7. Paper-ready description

### English

Hawaii monthly visitor arrivals are taken from the UHERO / DBEDT Tourism Data Warehouse official monthly series (statewide air-total, indicator VV101, market MM102, destination DI10), spanning 1989-01 to 2026-04. Hawaii normal-climate variables (monthly mean temperature, monthly total precipitation) come from NOAA NCEI Climate at a Glance statewide monthly time series 51 (Hawaii), with unit conversion from Fahrenheit to Celsius and from inches to millimeters; the monthly series covers 1991-01 to 2026-05. A parallel NOAA GHCN-Daily station-pool daily climate layer is available for station-level daily analysis, with TMAX, TMIN, TAVG, and PRCP per station per day; the default pool is 7 stations (Honolulu Intl, Hilo Intl, Kailua Kona Ke-Ahole, Kahului, Lihue WSO, Waikiki, Hilo 86A) and MULTIPLE relaxed coverage windows exist (modern32 1998-2025, relaxed 1989-2025, relaxed 1998-2023, relaxed 2000-2024, and additional annual-start rolling windows); if the GHCN daily layer is used, the specific window must be specified. The two climate frequencies are NOT bridged: the monthly ready panel (`hawaii_tourism_climate_ready_panel.csv`) merges monthly tourism with the NCEI statewide monthly climate series keyed by month, and does NOT use the GHCN daily series aggregated to monthly. No fill or interpolation is applied in any of the monthly tourism, monthly climate, daily climate, or ready-panel layers. No dedicated extreme-weather processed layer is available for Hawaii in the current materials.

### Chinese

夏威夷月度访客量取自 UHERO / DBEDT 旅游数据仓库官方月度系列（全州航空入境总量，indicator=VV101、market=MM102、destination=DI10），覆盖 1989-01 至 2026-04。夏威夷常态气候变量（月平均气温、月总降水量）取自 NOAA NCEI Climate at a Glance 全州月度时序 51（夏威夷），单位由华氏度转摄氏度、英寸转毫米；月度气候覆盖 1991-01 至 2026-05。另有一层并行的 NOAA GHCN-Daily 站点池日度气候层供站点级日度分析使用，包含 TMAX、TMIN、TAVG、PRCP 四个要素；默认站池为 7 个站（Honolulu Intl、Hilo Intl、Kailua Kona Ke-Ahole、Kahului、Lihue WSO、Waikiki、Hilo 86A），并存在多套放宽后的覆盖窗口（modern32 1998-2025、relaxed 1989-2025、relaxed 1998-2023、relaxed 2000-2024，以及多套按起年滚动的窗口）；若使用 GHCN 日度层，必须显式说明所用窗口。两种气候频率不互插：月度 ready 面板（`hawaii_tourism_climate_ready_panel.csv`）以月份为主键把月度旅游与 NCEI 全州月度气候合并，**不**采用 GHCN 日度按月聚合的版本。月度旅游、月度气候、日度气候、ready 面板四层均不补值、不插值。当前材料中夏威夷没有专属的极端天气处理层。
