# West Lake Scenic Area

> 样本: West Lake Scenic Area (西湖景区, Hangzhou, China) — 第二层·景区精细样本
> 角色: 中国大陆高颗粒度旅游场景，作为主文（港澳台主样本）的中国大陆对照/扩展，不进主文主回归
> 撰写依据: `.tmp/eight-sample-data-processing-audit-2026-06-29.md` §1.6、`.tmp/eight-sample-data-processing-matrix-2026-06-29.csv` (West Lake rows)、`docs/three-layer-sample-handoff.md` §3、`data_processed/climate/nasa_power_scenic_climate_summary.csv` (West Lake row)、`data_processed/climate/scenic_tourism_climate_month_panel_summary.csv` (West Lake row)、`data_processed/climate/hangzhou_open_weather_download_summary.csv` (9 sub-datasets)

---

## 1. Tourism data

- raw source: West Lake Scenic Area Management Committee (Hangzhou Municipal)
- raw carrier: HTML monthly page (`data_raw/westlake_tourism/pages/westlake_YYYY_MM_month.html`)
- raw frequency: monthly (sum of component-spot monthly passenger flow)
- current official processed files:
  - PRIMARY: `data_processed/tourism/westlake_scenic_spot_flow_month.csv`
  - CROSS-CHECK: `data_processed/tourism/westlake_scenic_spot_flow_quarter.csv`
- time range:
  - raw tourism window: 2016-01 to 2026-04 (106 monthly rows in the scenic panel)
  - strict analysis window: 2016-01 to 2025-12 (102 monthly rows; 2026 is truncated because only 4 months available)
  - quarterly cross-check: 259 rows
- metric_variant: `paid_scenic_spot_passenger_flow_sum` (各分景点月度客流的合计；不是杭州市总游客量)
- KEY POINT: 2026 raw data exists but the strict analysis window is truncated to 2025-12. The distinction between "raw tourism window" and "analysis window" MUST be stated.
- KEY POINT: from 2026-01 onward, "灵隐飞来峰" is not present in the parsed monthly tables; this composition change is preserved as-is. `component_count=8` 仍为 8, but the specific spot list may differ.

## 2. Climate data

- raw source (PRIMARY layer): NASA POWER (NASA Langley Research Center)
- alternative layer (NOT promoted): Hangzhou Open Data (Hangzhou Municipal) — local station-level sub-datasets (9 sub-datasets: 43826, 43827, 53193, 80057, 81021, 83117, 83228, 84252, 84411; covering 萧山/余杭/临安/西站 微站等本地站)
- raw carrier: JSON (NASA POWER Temporal Daily / Point API) + CSV (Hangzhou Open Data, sub-daily: hourly / 5-min / 10-min / irregular)
- raw frequency: daily (NASA POWER, gridded/reanalysis) + sub-daily (Hangzhou Open Data)
- current official processed files (PRIMARY):
  - `data_processed/climate/nasa_power_scenic_daily_weather.csv` (West Lake rows)
  - `data_processed/climate/nasa_power_scenic_monthly_weather.csv` (West Lake rows)
  - `data_processed/climate/scenic_tourism_climate_month_panel.csv` (West Lake rows; monthly tourism × monthly NASA POWER merged)
- variables (NASA POWER): T2M, T2M_MAX, T2M_MIN, PRECTOTCORR (point-based gridded/reanalysis at the scenic-area representative coordinate)
- time range: NASA POWER 1981-01-01 to 2026-06-08 (raw request window); `daily_valid_end` 2026-06-05; monthly `monthly_end` 2026-06
- Hangzhou Open Data sub-datasets are NOT promoted to a single West Lake series; the union does not cover the full 2016-01 to 2025-12 tourism window. No single sub-dataset covers the full window.
- coordinate source: UNESCO World Heritage Centre, West Lake Cultural Landscape of Hangzhou maps (https://whc.unesco.org/en/list/1334/maps/)
- KEY POINT: NASA POWER is gridded/reanalysis at the scenic-area representative coordinate (from UNESCO World Heritage Centre, West Lake Cultural Landscape of Hangzhou maps), NOT a local West Lake automatic weather station.

## 3. Extreme-weather data

- The current material does NOT have a dedicated extreme-weather processed layer for West Lake. Monthly extreme-day counts (hot 30°C / hot 35°C / cold 0°C / heavy rain 10mm / 25mm / 50mm) are derived from NASA POWER daily and embedded in `nasa_power_scenic_monthly_weather.csv`.
- Variables in the daily/monthly files include: T2M, T2M_MAX, T2M_MIN, PRECTOTCORR plus monthly extreme-day counts derived from daily values.
- State this explicitly: current materials do not have a dedicated extreme-weather processed layer; extreme-day counts are derived from the NASA POWER daily layer.

## 4. Processing steps

### 4.1 Tourism (monthly + quarterly)

- input: official West Lake scenic-area monthly HTML pages (`data_raw/westlake_tourism/pages/westlake_YYYY_MM_month.html`)
- intermediate: parse each month page; for each month extract the per-spot monthly passenger-flow table; sum across component spots
- output:
  - `data_processed/tourism/westlake_scenic_spot_flow_month.csv` (monthly; 106 rows; metric_variant = `paid_scenic_spot_passenger_flow_sum`)
  - `data_processed/tourism/westlake_scenic_spot_flow_quarter.csv` (quarterly cross-check; 259 rows)
- script: `scripts/data_fetch/fetch_westlake_scenic_spot_flow.py`

### 4.2 NASA POWER climate

- input: NASA POWER Temporal Daily / Point API at the West Lake representative coordinate (from UNESCO World Heritage Centre, West Lake Cultural Landscape of Hangzhou maps)
- intermediate: download daily JSON (`data_raw/nasa_power_scenic_climate/westlake_19810101_20260608.json`); keep 3-day TAVG/PRCP gaps as null
- output:
  - `data_processed/climate/nasa_power_scenic_daily_weather.csv` (West Lake rows of multi-sample shared file)
  - `data_processed/climate/nasa_power_scenic_monthly_weather.csv` (West Lake rows of multi-sample shared file; daily aggregated to monthly means + monthly extreme-day counts)
  - merge into `data_processed/climate/scenic_tourism_climate_month_panel.csv` (West Lake rows of multi-sample shared ready panel; monthly tourism × monthly NASA POWER)
- script: `scripts/data_fetch/fetch_nasa_power_scenic_climate.py`

### 4.3 Hangzhou Open Data (exploratory; NOT promoted)

- input: 9 Hangzhou Open Data sub-datasets (id: 43826, 43827, 53193, 80057, 81021, 83117, 83228, 84252, 84411), each at sub-daily frequency (hourly / 5-min / 10-min / irregular), with different time windows and station sets
- intermediate: per-sub-dataset, keep raw
- output: per-sub-dataset raw CSV under `data_raw/hangzhou_open_weather/downloads/`; download summary at `data_processed/climate/hangzhou_open_weather_download_summary.csv`. No cross-dataset merge into a single West Lake series. No aggregation to monthly.
- script: `scripts/data_fetch/fetch_hangzhou_open_weather.py`

## 5. Missingness / gap filling / aggregation

- Tourism: no fill; component-count changes preserved. `component_count=8` overall, but the specific spot list may differ across the window (2026-01 起"灵隐飞来峰"未出现在已解析月表中，原样保留，不补).
- NASA POWER: 3-day TAVG/PRCP gaps in the daily series are kept null (recorded as `missing_tavg_days=3`, `missing_prcp_days=3` in `nasa_power_scenic_climate_summary.csv`).
- Hangzhou Open Data: not promoted; no fill; per-sub-dataset raw kept; missing dates per sub-dataset left null.
- Aggregation:
  - daily → monthly: simple mean of daily values into monthly mean (NASA POWER)
  - daily → monthly extreme-day counts: hot30/hot35/cold0/heavy10/25/50 derived from daily values
  - monthly tourism × monthly NASA POWER: left-join on month key into `scenic_tourism_climate_month_panel.csv`
- summary files:
  - `data_processed/climate/nasa_power_scenic_climate_summary.csv` (per-sample climate source summary)
  - `data_processed/climate/scenic_tourism_climate_month_panel_summary.csv` (West Lake row: `tourism_month_start=2016-01`, `tourism_month_end=2026-04`, `tourism_observed_months=106`, `missing_months_between_start_end=18`, `complete_tourism_months=102`, `strict_core_panel_start=2016-01`, `strict_core_panel_end=2025-12`, `strict_core_panel_months=102`, `longest_strict_run_months=96`, `main_caveat=core_with_component_flag`)

## 6. Final processed files

| file | role | frequency | start | end | note |
|---|---|---|---|---|---|
| `data_processed/tourism/westlake_scenic_spot_flow_month.csv` | PRIMARY tourism monthly | monthly | 2016-01 | 2026-04 | 分景点月度客流加总（`metric_variant=paid_scenic_spot_passenger_flow_sum`）；raw tourism window 106 monthly rows；2026-01 起"灵隐飞来峰"未出现在已解析月表中，组成变化原样保留；strict analysis window 截到 2025-12 |
| `data_processed/tourism/westlake_scenic_spot_flow_quarter.csv` | cross-check tourism quarterly | quarterly | varies | varies | 同源季度加总，259 rows，supplementary |
| `data_processed/climate/nasa_power_scenic_daily_weather.csv` | PRIMARY climate daily | daily | 1981-01-01 | 2026-06-05 | 多样本共享文件，West Lake 行由 UNESCO 坐标的 NASA POWER daily 派生；3 天 TAVG/PRCP 缺口保留为 null |
| `data_processed/climate/nasa_power_scenic_monthly_weather.csv` | PRIMARY climate monthly | monthly | 1981-01 | 2026-06 | 多样本共享文件，West Lake 行由日值聚合为月均值与月极端日数 |
| `data_processed/climate/scenic_tourism_climate_month_panel.csv` | monthly tourism × climate ready panel | monthly | 2016-01 | 2026-04 | 多样本共享文件（5 景区样本）；West Lake 行由月度旅游与 NASA POWER 月度合并 |
| `data_processed/climate/scenic_tourism_climate_month_panel_summary.csv` | ready panel summary | per-sample | — | — | West Lake 行：`strict_core_panel_start=2016-01`、`strict_core_panel_end=2025-12`、`strict_core_panel_months=102`、`main_caveat=core_with_component_flag` |
| `data_processed/climate/scenic_tourism_climate_month_panel_missing_months.csv` | missing months list | per-month | — | — | 多样本共享文件；记录 ready panel 中旅游/气候的缺失月 |
| `data_processed/climate/nasa_power_scenic_climate_summary.csv` | per-sample climate source summary | per-sample | — | — | West Lake 行：来源 NASA POWER、UNESCO 坐标、1981-01-01 至 2026-06-05、16595 daily rows、3 TAVG + 3 PRCP 缺失日；`caveat` 显式说明"NASA POWER is an official NASA point API, but values represent source-native gridded/reanalysis data at a coordinate, not a local scenic-site automatic station" |
| `data_processed/climate/hangzhou_open_weather_download_summary.csv` | alternative layer download summary (NOT promoted) | per-sub-dataset | — | — | 9 个杭州开放数据子数据集（43826/43827/53193/80057/81021/83117/83228/84252/84411），频次为小时/5-min/10-min/不规则；未合并为单一 West Lake series |

## 7. Paper-ready description

**English.** West Lake monthly scenic-spot passenger flow is taken from the West Lake Scenic Area Management Committee (Hangzhou Municipal) official HTML monthly pages, summed across paid component spots (metric_variant = `paid_scenic_spot_passenger_flow_sum`). The raw tourism window is 2016-01 to 2026-04 (106 monthly rows in the scenic panel); the strict analysis window is 2016-01 to 2025-12 (102 monthly rows), truncated at 2025-12 because 2026 is incomplete (only 4 months available). From 2026-01 onward, "灵隐飞来峰" (Lingyin Feilai Peak) is not present in the parsed monthly tables; this composition change is preserved as-is, and the analysis window is truncated to 2025-12 to keep the component composition stable. Climate is from NASA POWER gridded/reanalysis daily data at the West Lake representative coordinate (from the UNESCO World Heritage Centre maps for the West Lake Cultural Landscape of Hangzhou), aggregated to monthly means and monthly extreme-day counts; this is NOT a local West Lake automatic weather station. Monthly extreme-day counts (hot 30/35 °C, cold 0 °C, heavy rain 10/25/50 mm) are derived from the NASA POWER daily layer. Hangzhou Open Data provides local station-level climate sub-datasets (rainfall, temperature, humidity, wind, etc.) at sub-daily frequencies for Xiaoshan / Yuhang / Lin'an / Hangzhou West Station micro-stations, but the union does not cover the full 2016-01 to 2025-12 tourism window and is not promoted to a single West Lake primary climate series.

**中文。** 西湖景区月度客流数据取自杭州西湖风景名胜区管理委员会的官方 HTML 月报，按各分景点月度客流加总（`metric_variant=paid_scenic_spot_passenger_flow_sum`）。原始数据范围 2016-01 至 2026-04（景区面板 106 行月度）；严格分析窗口 2016-01 至 2025-12（102 行月度），因 2026 年仅 4 个月不完整而截断。自 2026-01 起，"灵隐飞来峰"未出现在已解析月表中，该分景点组成变化原样保留，分析窗口截到 2025-12 以保持分景点组成稳定。气候取自 NASA POWER 网格/再分析日度数据（坐标源自 UNESCO 世界遗产中心 West Lake Cultural Landscape of Hangzhou 地图），按月聚合为月均值与月极端日数；该数据源并非本地自动气象站。月度极端日数（hot 30/35 °C、cold 0 °C、heavy rain 10/25/50 mm）由 NASA POWER 日度层派生。杭州开放数据提供萧山 / 余杭 / 临安 / 杭州西站等本地微站的站点级气候子数据集（降水、气温、湿度、风等），频次为亚日级，但尚未合成覆盖 2016-01 至 2025-12 完整西湖人旅窗口的单一序列，未升格为西湖人旅主气候层。
