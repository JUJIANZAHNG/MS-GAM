# Jiuzhaigou

## 1. Tourism data

- raw source: Jiuzhaigou Scenic Area official website (九寨沟景区官方网站)
- raw source page: `https://www.jiuzhai.com/news/number-of-tourists`
- raw carrier: HTML daily bulletin (parsed and flatened into `data_raw/jiuzhai_tourism/jiuzhai_arrivals_day_raw.csv`)
- raw frequency: daily (official)
- time range (raw): 2015-06-13 to 2026-06-02

Current official processed files (THREE layers + summary + audit):

- LAYER 1 (RAW, no fill): `data_processed/tourism/jiuzhai_arrivals_day.csv`
- LAYER 2 (BRIDGED, 32 single-day bridge rows outside closure windows): `data_processed/tourism/jiuzhai_arrivals_day_bridged.csv`
- LAYER 2 audit log: `data_processed/tourism/jiuzhai_arrivals_day_gap_fill_log.csv` (32-row log)
- LAYER 3 (MONTHLY ANALYSIS, sum of LAYER 2): `data_processed/tourism/jiuzhai_arrivals_month_bridged.csv`
- Processing change summary (raw vs bridged per month): `data_processed/tourism/jiuzhai_arrivals_processing_change_summary.csv`
- Closure-reopen audit (Markdown): `docs/jiuzhai_closure_reopen_audit.md`

KEY POINTS:

- LAYER 1 (raw) keeps official closure windows as null:
  - earthquake closure: 2017-08-09 to 2018-02
  - COVID closure: 2020-01-28 to 2020-03-30
- LAYER 2 (bridged) adds 32 isolated single-day bridge rows OUTSIDE the closure windows; each row is flagged in `imputed_flag` and recorded in `jiuzhai_arrivals_day_gap_fill_log.csv`. The 32 dates are listed in `docs/jiuzhai_closure_reopen_audit.md` §1. Some of these use `linear_interp_neighbor` (29 dates; script field `impute_method = local_linear_rounded`); some use `holiday_shape_scaled` (3 dates: 2016-05-01, 2016-05-02, 2016-05-03; script field `impute_method = holiday_shape_scaled_rounded`, donor window = 2017-05-01..2017-05-03 with 2017-05-04..2017-05-10 as the post-period reference).
- LAYER 3 (monthly) sums the bridged daily series; each month records observed vs expected tourism days, completeness ratio, and the number of bridged days (fields: `observed_tourism_days`, `expected_tourism_days`, `missing_days`, `completeness_ratio`, `n_imputed_days`, `has_imputed_days`, `is_tourism_month_observed`, `is_complete_tourism_month`).
- The closure-reopen audit (`docs/jiuzhai_closure_reopen_audit.md`) checks each of the 32 bridge dates against the official closure/reopen announcements and confirms none of the 32 bridge dates fall inside a confirmed official closure window. The four official closure/reopen anchor windows the audit cross-checks against are: 2017-08-08 earthquake closure, 2018-03-01 partial reopen, 2019-09-23 trial reopen, 2020-01-28 COVID closure, 2020-03-26 COVID partial reopen, 2021-09-28 full reopen. The four 2020-08-18..2020-08-21 bridge rows sit in the post-2020-03-31 reopening period and are explicitly tagged as "reopening-period short-gap bridge, retained but flagged".

## 2. Climate data

- raw source: NASA POWER (NASA Langley Research Center)
- raw source API: `https://power.larc.nasa.gov/api/temporal/daily/point`
- raw carrier: JSON (NASA POWER Temporal Daily / Point API, community=AG, parameters=T2M, T2M_MAX, T2M_MIN, PRECTOTCORR)
- raw frequency: daily (gridded/reanalysis at scenic-area representative coordinate)
- Jiuzhaigou coordinate: latitude 33.083330, longitude 103.916670
- coordinate source: UNESCO World Heritage Centre, Jiuzhaigou Valley maps (`https://whc.unesco.org/en/list/637/maps/`)
- time range: 1981-01-01 to 2026-06-05 (daily valid window per `nasa_power_scenic_climate_summary.csv`; raw JSON window 1981-01-01 to 2026-06-08)

Current official processed files:

- `data_processed/climate/nasa_power_scenic_daily_weather.csv` (Jiuzhaigou rows; multi-sample shared file; 16,595 daily rows for the slug)
- `data_processed/climate/nasa_power_scenic_monthly_weather.csv` (Jiuzhaigou rows; multi-sample shared file; 546 monthly rows for the slug, 545 complete-weather months)
- `data_processed/climate/scenic_tourism_climate_month_panel.csv` (Jiuzhaigou rows; merged with `jiuzhai_arrivals_month_bridged` tourism)
- `data_processed/climate/nasa_power_scenic_climate_summary.csv` (per-sample source summary; Jiuzhaigou row)
- Raw cache: `data_raw/nasa_power_scenic_climate/jiuzhaigou_19810101_20260608.json`

KEY POINT: NASA POWER is gridded/reanalysis at the Jiuzhaigou representative coordinate (from UNESCO World Heritage Centre, Jiuzhaigou Valley maps), NOT a local Jiuzhaigou automatic weather station. The `nasa_power_scenic_climate_summary.csv` Jiuzhaigou row explicitly carries this caveat.

## 3. Extreme-weather data

No dedicated extreme-weather processed layer for Jiuzhaigou. Monthly extreme-day counts (hot_day_30c_count, hot_day_35c_count, cold_day_0c_count, heavy_rain_10mm_count, heavy_rain_25mm_count, heavy_rain_50mm_count) are derived from the NASA POWER daily series and embedded as columns in `nasa_power_scenic_monthly_weather.csv` (Jiuzhaigou rows).

State explicitly: current materials do not have a dedicated extreme-weather processed layer for Jiuzhaigou; extreme-day counts are derived from the NASA POWER daily layer and stored in the monthly aggregation file.

## 4. Processing steps

### 4.1 Tourism LAYER 1 (raw)

- Script: `scripts/data_fetch/fetch_jiuzhai_arrivals.py`
- Input: HTML pages of `https://www.jiuzhai.com/news/number-of-tourists`, paginated by `start=0,20,40,...` (PAGE_SIZE=20)
- Parsing: per-row, parse the `共接待 N 人次` title with regex `共接待\s*([0-9,，]+)\s*人次` and the daily date from `td.list-date`; one row per bulletin
- Intermediate: pandas frame sorted by date, deduplicated on `date`, validated (no negative arrivals, no duplicates, monotonic decreasing)
- Output:
  - raw: `data_raw/jiuzhai_tourism/jiuzhai_arrivals_day_raw.csv`
  - processed LAYER 1: `data_processed/tourism/jiuzhai_arrivals_day.csv` (no fill)

### 4.2 Tourism LAYER 2 (bridged daily) and LAYER 3 (monthly) + change summary + bridge log

- Script: `scripts/data_fetch/bridge_jiuzhai_arrivals_day.py`
- Input: LAYER 1 (`jiuzhai_arrivals_day.csv`)
- Intermediate processing:
  - expand the raw series to a continuous daily date range (`pd.date_range(min, max, freq="D")`); left-merge raw onto the full range
  - compute `arrivals_interp` by `pandas.Series.interpolate(method="linear", limit_direction="both")` (used only to extract anchor-free float values for the 29 `local_linear_rounded` rows; the result is rounded to an integer with `round_count`)
  - for each of the 29 `LOCAL_LINEAR_GAP_DATES`, take the immediately previous and next observed rows as anchors; emit a fill row with `imputed_flag=1`, `impute_method="local_linear_rounded"`, `gap_class="short_local_gap"`, and the two anchor `(date, arrivals)` pairs
  - for each of the 3 `HOLIDAY_SHAPE_SCALED_GAP_DATES` (2016-05-01, 2016-05-02, 2016-05-03), take the donor 2017-05-01..2017-05-03 (Labour Day) holiday values and rescale them by `target_post_mean_2016_0504_0510 / donor_post_mean_2017_0504_0510`; emit a fill row with `imputed_flag=1`, `impute_method="holiday_shape_scaled_rounded"`, `gap_class="holiday_gap_scaled_from_2017_mayday"`
  - bridge the 29 + 3 = 32 rows to the observed frame; sort by date; assert no duplicate dates
- Output:
  - LAYER 2 bridged daily: `data_processed/tourism/jiuzhai_arrivals_day_bridged.csv` (32 extra rows; each `imputed_flag=1`)
  - bridge log (one row per bridged day): `data_processed/tourism/jiuzhai_arrivals_day_gap_fill_log.csv` (32 rows)
  - LAYER 3 monthly: `data_processed/tourism/jiuzhai_arrivals_month_bridged.csv` (sum of bridged daily per month, with `observed_tourism_days / expected_tourism_days / missing_days / completeness_ratio / n_imputed_days / has_imputed_days / is_tourism_month_observed / is_complete_tourism_month`)
  - raw monthly (intermediate, embedded in the change summary): from raw daily, with the same month-aggregate schema
  - processing change summary: `data_processed/tourism/jiuzhai_arrivals_processing_change_summary.csv` (per-month raw vs bridged delta, with `arrivals_added_by_imputation`, `observed_days_added`, `missing_days_reduced`, `completeness_ratio_delta`, and a `change_status` of `already_complete / incomplete_to_complete / partial_gain_still_incomplete / still_absent / no_change`)

### 4.3 NASA POWER climate

- Script: `scripts/data_fetch/fetch_nasa_power_scenic_climate.py`
- Input: NASA POWER daily point API for the Jiuzhaigou representative coordinate (latitude 33.083330, longitude 103.916670, from UNESCO World Heritage Centre, Jiuzhaigou Valley maps)
- Intermediate processing:
  - call `https://power.larc.nasa.gov/api/temporal/daily/point?parameters=T2M,T2M_MAX,T2M_MIN,PRECTOTCORR&community=AG&...&start=YYYYMMDD&end=YYYYMMDD&format=JSON`
  - flatten the JSON's `properties.parameter` tree into one row per (slug, date) with `tavg_c`, `tmax_c`, `tmin_c`, `prcp_mm` (from `T2M`, `T2M_MAX`, `T2M_MIN`, `PRECTOTCORR` respectively); treat the API's `header.fill_value` sentinel as null
  - aggregate to monthly per (slug, month) with `mean_tavg_c / mean_tmax_c / mean_tmin_c / total_prcp_mm / max_daily_prcp_mm`, plus the derived extreme-day counts (`hot_day_30c_count`, `hot_day_35c_count`, `cold_day_0c_count`, `heavy_rain_10mm_count`, `heavy_rain_25mm_count`, `heavy_rain_50mm_count`), per-variable day counts (`n_tavg_days`, `n_tmax_days`, `n_tmin_days`, `n_prcp_days`), and `is_complete_weather_month`
- Output:
  - daily: `data_processed/climate/nasa_power_scenic_daily_weather.csv` (multi-sample shared file with Jiuzhaigou rows)
  - monthly: `data_processed/climate/nasa_power_scenic_monthly_weather.csv` (multi-sample shared file with Jiuzhaigou rows)
  - summary: `data_processed/climate/nasa_power_scenic_climate_summary.csv` (per-sample source summary)
  - merged panel: `data_processed/climate/scenic_tourism_climate_month_panel.csv` (Jiuzhaigou rows; tourism = `jiuzhai_arrivals_month_bridged`, climate = NASA POWER monthly)

## 5. Missingness / gap filling / aggregation

- LAYER 1 (raw): no fill; closure windows (earthquake 2017-08-09 to 2018-02; COVID 2020-01-28 to 2020-03-30) stay null.
- LAYER 2 (bridged): 32 single-day bridge rows added OUTSIDE closure windows; closure windows themselves are NOT bridged; the bridge scope is the isolated single day only.
- LAYER 3 (monthly): no fill; closure-window days remain null at the daily layer, so monthly values under-count closures.
- NASA POWER daily: 3-day TAVG/PRCP gaps in the daily series kept null (per `nasa_power_scenic_climate_summary.csv` Jiuzhaigou row, `missing_tavg_days=3`, `missing_prcp_days=3`).
- Aggregations:
  - daily -> monthly for tourism: sum of LAYER 2 (bridged) daily into monthly `visitor_arrivals`; observed-day / expected-day counts are also computed.
  - daily -> monthly for climate: simple mean of daily T2M, T2M_MAX, T2M_MIN into `mean_tavg_c / mean_tmax_c / mean_tmin_c`; sum of daily `PRECTOTCORR` into `total_prcp_mm`; max of daily `PRECTOTCORR` into `max_daily_prcp_mm`; monthly extreme-day counts (hot30/hot35/cold0/heavy10/25/50) derived from daily values.

## 6. Final processed files

| file | role | frequency | start | end | note |
|---|---|---|---|---|---|
| `data_processed/tourism/jiuzhai_arrivals_day.csv` | RAW (no fill; closure windows null) | daily | 2015-06-13 | 2026-06-02 | LAYER 1; closure windows (earthquake 2017-08-09 to 2018-02; COVID 2020-01-28 to 2020-03-30) left as null |
| `data_processed/tourism/jiuzhai_arrivals_day_bridged.csv` | BRIDGED (32 single-day bridge rows) | daily | 2015-06-13 | 2026-06-02 | LAYER 2; 32 isolated single-day bridge rows added OUTSIDE closure windows; each row flagged in `imputed_flag` |
| `data_processed/tourism/jiuzhai_arrivals_day_gap_fill_log.csv` | 32-row bridge log | per-bridge-day | — | — | One row per bridged day; records `impute_method`, `gap_class`, anchor dates, `interpolated_float`, `arrivals_filled` |
| `data_processed/tourism/jiuzhai_arrivals_month_bridged.csv` | MONTHLY ANALYSIS layer (sum of bridged daily) | monthly | 2015-06 | 2026-06 | LAYER 3; observed / expected days, completeness ratio, bridged-day count per month |
| `data_processed/tourism/jiuzhai_arrivals_processing_change_summary.csv` | raw vs bridged per-month comparison | monthly | 2015-06 | 2026-06 | Per-month raw vs bridged delta with `change_status` |
| `docs/jiuzhai_closure_reopen_audit.md` | closure-reopen audit (Markdown file) | per-bridge-day | 2017-08-08 | 2021-09-28 | Audit file; confirms none of the 32 bridge dates fall inside a confirmed official closure window |
| `data_processed/climate/nasa_power_scenic_daily_weather.csv` | PRIMARY climate daily (multi-sample shared file; Jiuzhaigou rows owned by `slug = "jiuzhaigou"`) | daily | 1981-01-01 | 2026-06-05 | T2M, T2M_MAX, T2M_MIN, PRECTOTCORR at UNESCO Jiuzhaigou coordinate |
| `data_processed/climate/nasa_power_scenic_monthly_weather.csv` | PRIMARY climate monthly (multi-sample shared file; Jiuzhaigou rows owned by `slug = "jiuzhaigou"`) | monthly | 1981-01 | 2026-06 | Monthly mean/max/min, total/max prcp, plus derived extreme-day counts |
| `data_processed/climate/scenic_tourism_climate_month_panel.csv` | monthly tourism x climate ready panel (5 scenic samples) | monthly | per sample | per sample | Jiuzhaigou rows owned by `slug = "jiuzhaigou"`; tourism = `jiuzhai_arrivals_month_bridged`; climate = NASA POWER monthly |
| `data_processed/climate/scenic_tourism_climate_month_panel_summary.csv` | summary | per-sample | — | — | Jiuzhaigou row: `tourism_metric_label = 官网日度游客接待量月度加总`, `unit_type = scenic_area`, `quality_tier = core_daily_with_closure_gaps`, `main_caveat = 日度官网数据按月加总；自然日缺口多为景区关闭或官网停更段，不能当作零客流自动填补。` |
| `data_processed/climate/scenic_tourism_climate_month_panel_missing_months.csv` | per-sample missing-months manifest | per-month | per sample | per sample | Jiuzhaigou rows enumerate months that are not complete in the strict panel |
| `data_processed/climate/nasa_power_scenic_climate_summary.csv` | per-sample climate source summary | per-sample | 1981-01-01 | 2026-06-08 | Jiuzhaigou row carries caveat: "NASA POWER is an official NASA point API, but values represent source-native gridded/reanalysis data at a coordinate, not a local scenic-site automatic station." |

## 7. Paper-ready description

### English (paragraph)

Jiuzhaigou daily visitor arrivals come from the Jiuzhaigou Scenic Area official daily bulletin and are organised into three explicit layers. The raw series (`jiuzhai_arrivals_day.csv`, 2015-06-13 to 2026-06-02) keeps the official closure windows as null: the earthquake closure 2017-08-09 to 2018-02 and the COVID closure 2020-01-28 to 2020-03-30. A separate bridged daily layer (`jiuzhai_arrivals_day_bridged.csv`) adds 32 isolated single-day bridge rows outside those two closure windows; each row is flagged in `imputed_flag` and recorded in `jiuzhai_arrivals_day_gap_fill_log.csv`, and the closures themselves are NOT bridged. A closure-reopen audit (`docs/jiuzhai_closure_reopen_audit.md`) checks each bridge date against the official closure/reopen announcements and confirms none of the 32 bridge dates falls inside a confirmed official closure window. The monthly analysis series (`jiuzhai_arrivals_month_bridged.csv`) is the sum of the bridged daily series and records observed vs expected tourism days, completeness ratio, and bridged-day count per month. Climate for Jiuzhaigou is taken from NASA POWER gridded/reanalysis daily data at the Jiuzhaigou representative coordinate (from UNESCO World Heritage Centre, Jiuzhaigou Valley maps), aggregated to monthly means and monthly extreme-day counts; this is NOT a local Jiuzhaigou automatic weather station.

### Chinese (paragraph)

九寨沟日度游客接待量取自九寨沟景区官方日度公告，并按三层文件分别保存。raw 系列（`jiuzhai_arrivals_day.csv`，2015-06-13 至 2026-06-02）保留官方闭园窗口为缺失：地震闭园 2017-08-09 至 2018-02、疫情闭园 2020-01-28 至 2020-03-30。另有 bridged 日度层（`jiuzhai_arrivals_day_bridged.csv`）在两个闭园窗口之外对 32 个孤立单日做桥接，逐行 `imputed_flag` 标记并记入 `jiuzhai_arrivals_day_gap_fill_log.csv`；闭园窗口本身不桥接。闭园/复园审计（`docs/jiuzhai_closure_reopen_audit.md`）逐日核对 32 个桥接日是否落在官方公告闭园窗口内，确认均未撞线。月度分析层（`jiuzhai_arrivals_month_bridged.csv`）由 bridged 日度序列按月加总得到，记录每月观测 / 应到天数、完整度与填补天数。九寨沟气候取自 NASA POWER 网格 / 再分析日度数据（坐标源自 UNESCO 世界遗产中心九寨沟山谷地图），按月聚合为月均值与月极端日数；该数据并非九寨沟本地自动气象站观测。
