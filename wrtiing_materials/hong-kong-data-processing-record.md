# Hong Kong

## 1. Tourism data

- Raw source institution: Hong Kong Tourism Board (HKTB)
- Raw carrier (format): XLS (one file per month; `data_raw/hong_kong_tourism/vas_*.xls`); the 2002-01 to 2012-12 portion is published as official PDF (parsed by `pdftotext -layout`)
- Raw frequency: monthly
- Current official processed file: `data_processed/tourism/hong_kong_visitor_arrivals_month.csv`
- Time range: 2002-01 to 2026-04 (raw series 2002-01..2012-12 from PDF, 2013-01..2026-04 from XLS)

## 2. Climate data

HK has multiple climate layers; the primary and three alternatives are listed below.

- Raw source institution: Hong Kong Observatory (HKO)
- Raw carrier (format):
  - Primary (monthly): CSV via HKO OpenData API (`data.weather.gov.hk`) for HKO Headquarters mean / max / min temperature and total rainfall (daily), plus event-level `.dat` warning files
  - Station-level daily + monthly: HKO automatic weather station (AWS) XML/JSON (`cis/aws/individual_day/daily_{station}_{year}.xml`)
  - HKO TC impact tables: `TC_Impact_Data_HKO.xlsx` + metadata PDF
- Raw frequency: daily (HKO HQ / station), event-level (warning `.dat`), monthly (HKO impact workbook by TC)
- Current official processed files:
  - Primary (monthly): `hong_kong_hko_monthly_weather.csv`
  - Station-level (alternative): `hong_kong_station_daily_weather_panel.csv` + `hong_kong_station_monthly_weather_panel.csv` (+ `hong_kong_station_catalog.csv`)
  - HKO TC impact (supplementary): `hong_kong_tc_impact_signal_events.csv` + `hong_kong_tc_impact_rainfall_events.csv` + `hong_kong_tc_impact_other_met_events.csv`
- Time range:
  - HKO monthly HQ series: 1884-01 to 2026-05
  - HKO daily HQ series: 1884-03-01 to 2026-04-30 (per `tourism-management-data-feasibility.md` parse)
  - Station daily panel: 1984-10-02 to 2026-05-31
  - HKO TC impact workbook: 1946 (earliest TC) to 2024 (latest fetched)

## 3. Extreme-weather data

HK has 8 warning families: cold weather, fire danger, frost, very hot weather, landslip, black/red/amber rainstorm (rainstorm family), tropical cyclone warning signal, thunderstorm.

- Event-level: `data_processed/climate/hong_kong_warning_events.csv` (parsed from HKO `cold.dat` / `fire.dat` / `frost.dat` / `hot.dat` / `lslip.dat` / `nf.dat` / `rstorm.dat` / `tc.dat` / `thunder.dat` / `tcname.dat`)
- Monthly aggregation: `data_processed/climate/hong_kong_warning_event_month.csv` (event count + total warning hours per family, grouped by warning start month; plus derived `tc_signal_8plus_*` and `rainstorm_red_black_*`)
- Earliest warning .dat series start points: `cold.dat` 1999-12-19 03:10; `thunder.dat` 1967-04-03 22:00; `fire.dat` 1995-01-01 06:15 (other families in the same HKO warning .dat set)
- Monthly aggregation: full month grid filled via `pd.date_range` from min to max month; missing family-month combinations filled with `0` / `0.0`

## 4. Processing steps

Tourism input -> intermediate processing -> output:
- Input: HKTB PartnerNet `Visitor Arrival Statistics` (VAS) index page; 2002-2012 official PDF; 2013+ official XLS.
- Intermediate: parse index for Excel and PDF links; for each XLS open `Tables 1 & 2`, find `Total` row, locate the current-month arrivals column by the 100.0% column anchor, and record arrivals + previous-year same-month + YoY; for each PDF run `pdftotext -layout -f 1 -l 3` and extract the `Total` row's second numeric value.
- Output: `data_processed/tourism/hong_kong_visitor_arrivals_month.csv`.
- Fetch script: `scripts/data_fetch/fetch_hong_kong_hktb_arrivals.py`.

Climate input -> intermediate processing -> output:
- Input: HKO OpenData API daily CSV (mean / max / min temperature, total rainfall) for HKO HQ; HKO warning .dat files; HKO stn.htm station catalog; HKO AWS daily JSON/XML per station per year; HKO `TC_Impact_Data_HKO.xlsx`.
- Intermediate: parse HKO daily CSV (skip 2 header rows), build daily weather frame, then group by `month` to compute monthly mean / max / min temperature and monthly total rainfall, plus per-month `*_valid_days`, `*_unavailable_days`, `trace_rainfall_days`; parse 9 warning .dat files into a single event-level frame with `start_datetime`, `end_datetime`, `duration_hours`, then pivot to `*_event_count` and `*_total_warning_hours` by `start_month`; build station catalog from `stn.htm`, probe each panel station's supported elements, then fetch AWS JSON for each (station, year) pair from the station's `start_year` to current year, and aggregate station-daily to station-monthly; parse `TC_Impact_Data_HKO.xlsx` into three separate sheets (`Signal Data`, `Rainfall`, `Other Met Data`).
- Output: 10 processed files under `data_processed/climate/` (HKO daily, HKO monthly, warning events, warning month, three TC impact files, station catalog, station daily panel, station monthly panel) plus a per-file summary at `hong_kong_climate_summary.csv`.
- Fetch script: `scripts/data_fetch/fetch_hong_kong_climate.py`.

## 5. Missingness / gap filling / aggregation

- HKO HQ daily series: `***` / `N/A` / `NR` values are kept as null in the daily table; the `*_unavailable_days` flag counts them per month (logged in `hong_kong_hko_daily_weather.csv` and aggregated to `hong_kong_hko_monthly_weather.csv`). No temporal interpolation is applied in the processed layer.
- Trace rainfall is coded as `0.0 mm` with the `rainfall_trace_flag` retained; the monthly aggregation note field reads `aggregated from official HKO daily station series; rainfall Trace is coded as 0.0 mm with trace flag retained`.
- Monthly climate: no gap fill; `*_valid_days` and `*_unavailable_days` columns are reported per month.
- Station daily panel: missing station-days remain missing in the station panel; station-monthly aggregation uses `sum(min_count=1)` for rainfall and `mean()` for temperature, so all-null months return null.
- Warning events: no fill on warning events (event absence is meaningful); the monthly aggregation expands to a full month grid via `pd.date_range` and fills missing family-month cells with `0` / `0.0`.
- TC impact workbook: no fill; `times_of_passages` cells that contain footnote characters (e.g. `&`) are kept in `times_of_passages_raw` and not coerced to numeric.
- Tourism series: HKTB `2017-10` is missing in the current official page list; the script does not impute, it only writes the missing month to `data_raw/hong_kong_tourism/hong_kong_hktb_arrivals_missing_months.csv`.

## 6. Final processed files

| file | role | frequency | start | end | note |
|---|---|---|---|---|---|
| `data_processed/tourism/hong_kong_visitor_arrivals_month.csv` | Tourism (dependent variable) | monthly | 2002-01 | 2026-04 | HKTB VAS monthly XLS/PDF merged into a single monthly series; 2017-10 flagged as missing in raw |
| `data_processed/climate/hong_kong_hko_monthly_weather.csv` | Climate (primary monthly) | monthly | 1884-01 | 2026-05 | Aggregated from HKO HQ daily CSV; `aggregation_note` records Trace -> 0.0 mm rule |
| `data_processed/climate/hong_kong_hko_daily_weather.csv` | Climate (primary daily) | daily | 1884-03-01 | 2026-04-30 | HKO HQ daily mean / max / min temperature and total rainfall |
| `data_processed/climate/hong_kong_station_daily_weather_panel.csv` | Climate (station-level, alternative) | daily | 1984-10-02 | 2026-05-31 | HKO AWS station panel; not the primary regression source |
| `data_processed/climate/hong_kong_station_monthly_weather_panel.csv` | Climate (station-level, alternative) | monthly | 1984-10 | 2026-05 | Monthly aggregation of station daily panel |
| `data_processed/climate/hong_kong_station_catalog.csv` | Climate (station metadata) | catalog | n/a | n/a | Parsed from HKO `stn.htm`; active + terminated stations with `is_panel_station` flag |
| `data_processed/climate/hong_kong_warning_events.csv` | Extreme weather (event-level) | event | 1947+ (varies by family) | 2026-05 | 8 warning families + Strong Monsoon Signal from HKO `.dat` |
| `data_processed/climate/hong_kong_warning_event_month.csv` | Extreme weather (monthly aggregation) | monthly | varies by family | 2026-05 | Counts and total warning hours by warning start month; full month grid filled with 0 / 0.0 |
| `data_processed/climate/hong_kong_tc_impact_signal_events.csv` | TC impact (supplementary) | event | 1988-05-31 (Signal sheet) | 2024-11-18 (latest fetched) | `Signal Data` sheet of `TC_Impact_Data_HKO.xlsx` |
| `data_processed/climate/hong_kong_tc_impact_rainfall_events.csv` | TC impact (supplementary) | event | 1988 (Rainfall sheet) | 2024 | `Rainfall` sheet; some rows keep `times_of_passages_raw='&'` |
| `data_processed/climate/hong_kong_tc_impact_other_met_events.csv` | TC impact (supplementary) | event | 1988 (Other Met sheet) | 2024 | `Other Met Data` sheet (pressure, surge, sea level) |
| `data_processed/climate/hong_kong_climate_summary.csv` | Per-file summary | per-file | n/a | n/a | One row per output file with row count, start, end, and notes |
| `data_processed/climate/hong_kong_extreme_climate_month_demo.csv` | Demo join (NOT primary) | monthly demo | depends on source joins | depends on source joins | (demo, not the primary analysis file) |

## 7. Paper-ready description

Hong Kong monthly visitor arrivals come from the Hong Kong Tourism Board's monthly Visitor Arrival Statistics exports (XLS from 2013-01 onward, PDF from 2002-01 to 2012-12), consolidated into a single monthly series covering 2002-01 to 2026-04; 2017-10 is a publicly known gap and is left as missing. The primary normal-climate layer is the HKO Headquarters official daily series (mean / max / min temperature and total rainfall) obtained from the HKO OpenData API and aggregated to monthly; trace rainfall is encoded as 0.0 mm with the trace flag retained, and per-month `*_valid_days` / `*_unavailable_days` columns are preserved. Extreme-weather variables come from the HKO warning database covering eight families (cold, fire, frost, hot, landslip, rainstorm, tropical cyclone, thunderstorm), with event-level records aggregated to monthly counts and total warning hours. A station-level daily/monthly panel built from HKO automatic weather station exports and a separate HKO tropical cyclone impact workbook (Signal / Rainfall / Other Met sheets) are kept as an alternative climate layer and a supplementary TC impact layer, respectively. The `hong_kong_extreme_climate_month_demo.csv` file is a join demo and is not the primary analysis file.

香港月度访港旅客数据来自香港旅游发展局月度访港旅客统计（VAS）月度发布物（2013-01 起为 XLS，2002-01 至 2012-12 为 PDF），合并为单一份月度序列，覆盖 2002-01 至 2026-04；2017-10 为已知公开缺口，保留为缺失。常态气候主层为香港天文台总部站官方日度系列（平均 / 最高 / 最低气温与总降水量），由 HKO OpenData API 获取并按月聚合；微量（Trace）降水量按 0.0 mm 编码并保留 trace 标志，每月同时保留 `*_valid_days` 与 `*_unavailable_days` 列。极端天气变量取自香港天文台预警数据库，覆盖八族预警（寒冷、火灾、霜冻、酷热、山泥倾泻、暴雨、热带气旋、雷暴），事件级记录按月开始月聚合成月度次数与累计预警小时。另有基于 HKO 自动气象站导出的站点级日度与月度面板作为气候层的备选，以及一份独立的 HKO 热带气旋影响工作簿（Signal / Rainfall / Other Met 三张表）作为 TC 影响补充层。`hong_kong_extreme_climate_month_demo.csv` 是 join 演示文件，不作为主分析文件。
