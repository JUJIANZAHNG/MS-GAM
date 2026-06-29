# Gulangyu

> 样本: Gulangyu (鼓浪屿, Xiamen, China) — 第二层·景区精细样本
> 角色: 中国大陆高颗粒度旅游场景，作为主文（港澳台主样本）的中国大陆对照/扩展，不进主文主回归
> 撰写依据: `.tmp/eight-sample-data-processing-audit-2026-06-29.md` §1.4、`.tmp/eight-sample-data-processing-matrix-2026-06-29.csv` (Gulangyu rows)、`docs/three-layer-sample-handoff.md` §3、`data_processed/climate/nasa_power_scenic_climate_summary.csv` (Gulangyu row)、`data_processed/climate/scenic_tourism_climate_month_panel_summary.csv` (Gulangyu row)

---

## 1. Tourism data

- raw source: Gulangyu Scenic Area (Xiamen Municipal) — 鼓浪屿景区官方月报
- raw carrier: HTML monthly page (`data_raw/gulangyu_tourism/`)
- raw frequency: monthly (multiple `metric_variant` co-exist, primarily 上岛游客总量, plus 景点接待量 etc.)
- current official processed file:
  - PRIMARY: `data_processed/tourism/gulangyu_tourism_month.csv`
- time range: 2015-08 to 2026-04 (106 monthly rows in the scenic panel; 17 months are publicly confirmed official data gaps)
- KEY POINT: 2019-08 to 2020-11 is a publicly confirmed official data gap; the processed series keeps this window as null and does NOT fill it. The remaining missing months are also left as null.

## 2. Climate data

- raw source: NASA POWER (NASA Langley Research Center)
- raw carrier: JSON (NASA POWER Temporal Daily / Point API)
- raw frequency: daily (gridded/reanalysis at scenic-area representative coordinate)
- current official processed files (PRIMARY):
  - `data_processed/climate/nasa_power_scenic_daily_weather.csv` (Gulangyu rows)
  - `data_processed/climate/nasa_power_scenic_monthly_weather.csv` (Gulangyu rows)
  - `data_processed/climate/scenic_tourism_climate_month_panel.csv` (Gulangyu rows; monthly tourism × monthly NASA POWER merged)
- variables (NASA POWER): T2M, T2M_MAX, T2M_MIN, PRECTOTCORR (point-based gridded/reanalysis at the scenic-area representative coordinate)
- time range: NASA POWER 1981-01-01 to 2026-06-08 (raw request window); `daily_valid_end` 2026-06-05; monthly `monthly_end` 2026-06
- coordinate source: UNESCO World Heritage Centre, Kulangsu maps (https://whc.unesco.org/en/list/1541/maps/)
- KEY POINT: NASA POWER is gridded/reanalysis at the Gulangyu representative coordinate (from UNESCO World Heritage Centre, Kulangsu maps), NOT a local Gulangyu automatic weather station.

## 3. Extreme-weather data

- The current materials do NOT have a dedicated extreme-weather processed layer for Gulangyu. Monthly extreme-day counts (hot 30°C / hot 35°C / cold 0°C / heavy rain 10mm / 25mm / 50mm) are derived from the NASA POWER daily layer and embedded in `data_processed/climate/nasa_power_scenic_monthly_weather.csv`.
- Variables in the daily/monthly files include: T2M, T2M_MAX, T2M_MIN, PRECTOTCORR plus monthly extreme-day counts derived from daily values.
- State this explicitly: current materials do not have a dedicated extreme-weather processed layer; extreme-day counts are derived from the NASA POWER daily layer.

## 4. Processing steps

### 4.1 Tourism (monthly)

- input: official Gulangyu scenic-area monthly HTML pages (`data_raw/gulangyu_tourism/`)
- intermediate: parse each month page; preserve multiple `metric_variant` (上岛游客总量 / 景点接待量 etc.); keep 2019-08 to 2020-11 as null (publicly confirmed official data gap)
- output:
  - `data_processed/tourism/gulangyu_tourism_month.csv` (monthly; 106 rows in scenic panel; 17 months publicly confirmed null)
- script: `scripts/data_fetch/fetch_gulangyu_tourism_monthly.py`

### 4.2 NASA POWER climate

- input: NASA POWER Temporal Daily / Point API at the Gulangyu representative coordinate (from UNESCO World Heritage Centre, Kulangsu maps)
- intermediate: download daily JSON (`data_raw/nasa_power_scenic_climate/gulangyu_19810101_20260608.json`); keep 3-day TAVG/PRCP gaps as null
- output:
  - `data_processed/climate/nasa_power_scenic_daily_weather.csv` (Gulangyu rows of multi-sample shared file)
  - `data_processed/climate/nasa_power_scenic_monthly_weather.csv` (Gulangyu rows of multi-sample shared file; daily aggregated to monthly means + monthly extreme-day counts)
  - merge into `data_processed/climate/scenic_tourism_climate_month_panel.csv` (Gulangyu rows of multi-sample shared ready panel; monthly tourism × monthly NASA POWER)
- script: `scripts/data_fetch/fetch_nasa_power_scenic_climate.py`

## 5. Missingness / gap filling / aggregation

- Tourism: 2019-08 to 2020-11 is a publicly confirmed official data gap; left as null; no fill. Other missing months are also left as null (no interpolation, no bridging).
- NASA POWER: 3-day TAVG/PRCP gaps in the daily series are kept null (recorded as `missing_tavg_days=3`, `missing_prcp_days=3` in `nasa_power_scenic_climate_summary.csv`).
- Aggregations:
  - daily → monthly: simple mean of daily values into monthly mean (NASA POWER)
  - daily → monthly extreme-day counts: hot30/hot35/cold0/heavy10/25/50 derived from daily values
  - monthly tourism × monthly NASA POWER: left-join on month key into `scenic_tourism_climate_month_panel.csv`
- summary files:
  - `data_processed/climate/nasa_power_scenic_climate_summary.csv` (per-sample climate source summary)
  - `data_processed/climate/scenic_tourism_climate_month_panel_summary.csv` (Gulangyu row: `tourism_month_start=2015-08`, `tourism_month_end=2026-04`, `tourism_observed_months=112`, `missing_months_between_start_end=17`, `complete_tourism_months=106`, `strict_core_panel_start=2015-08`, `strict_core_panel_end=2026-04`, `strict_core_panel_months=106`, `longest_strict_run_months=37` (2023-04 to 2026-04), `main_caveat=core_with_gap`)

## 6. Final processed files

| file | role | frequency | start | end | note |
|---|---|---|---|---|---|
| `data_processed/tourism/gulangyu_tourism_month.csv` | PRIMARY tourism monthly | monthly | 2015-08 | 2026-04 | 鼓浪屿景区官方月报；多 `metric_variant` 并存（"上岛游客总量" / "景点接待量" 等）；2019-08 至 2020-11 为已确认公开官方缺口，保留为 null；106 monthly rows in scenic panel, 17 months publicly confirmed null |
| `data_processed/climate/nasa_power_scenic_daily_weather.csv` | PRIMARY climate daily | daily | 1981-01-01 | 2026-06-05 | 多样本共享文件（5 景区样本：Gulangyu / West Lake / Jiuzhaigou / Huangshan / Sanya），Gulangyu 行由 UNESCO 坐标的 NASA POWER daily 派生；3 天 TAVG/PRCP 缺口保留为 null |
| `data_processed/climate/nasa_power_scenic_monthly_weather.csv` | PRIMARY climate monthly | monthly | 1981-01 | 2026-06 | 多样本共享文件，Gulangyu 行由日值聚合为月均值与月极端日数（hot30/hot35/cold0/heavy10/25/50 等） |
| `data_processed/climate/scenic_tourism_climate_month_panel.csv` | monthly tourism × climate ready panel | monthly | 2015-08 | 2026-04 | 多样本共享文件（5 景区样本）；Gulangyu 行由月度旅游与 NASA POWER 月度合并 |
| `data_processed/climate/scenic_tourism_climate_month_panel_summary.csv` | ready panel summary | per-sample | — | — | Gulangyu 行：`tourism_month_start=2015-08`、`tourism_month_end=2026-04`、`tourism_observed_months=112`、`missing_months_between_start_end=17`、`complete_tourism_months=106`、`strict_core_panel_months=106`、`longest_strict_run_months=37`、`main_caveat=core_with_gap` |
| `data_processed/climate/scenic_tourism_climate_month_panel_missing_months.csv` | missing months list | per-month | — | — | 多样本共享文件；记录 ready panel 中旅游/气候的缺失月 |
| `data_processed/climate/nasa_power_scenic_climate_summary.csv` | per-sample climate source summary | per-sample | — | — | Gulangyu 行：来源 NASA POWER、UNESCO 坐标、1981-01-01 至 2026-06-05、16595 daily rows、3 TAVG + 3 PRCP 缺失日；`caveat` 显式说明"NASA POWER is an official NASA point API, but values represent source-native gridded/reanalysis data at a coordinate, not a local scenic-site automatic station" |

## 7. Paper-ready description

**English.** Gulangyu monthly visitor arrivals come from the Gulangyu Scenic Area (Xiamen Municipal) official monthly bulletin; the processed series keeps 2019-08 to 2020-11 as null because this window is a publicly confirmed official data gap, and no interpolation or bridging is applied (the remaining missing months are also left as null). The raw series covers 2015-08 to 2026-04 with 106 monthly rows in the scenic panel and 17 publicly confirmed null months. Multiple `metric_variant` values co-exist in the parsed pages (上岛游客总量 / 景点接待量 etc.); the primary analysis uses the 上岛游客总量 variant. Climate is from NASA POWER gridded/reanalysis daily data at the Gulangyu representative coordinate (from the UNESCO World Heritage Centre maps for Kulangsu), aggregated to monthly means and monthly extreme-day counts; this is NOT a local Gulangyu automatic weather station. The 3-day TAVG/PRCP gaps in the NASA POWER daily series are kept null. Monthly extreme-day counts (hot 30/35 °C, cold 0 °C, heavy rain 10/25/50 mm) are derived from the NASA POWER daily layer, since the current materials do not include a dedicated extreme-weather processed layer for Gulangyu.

**中文。** 鼓浪屿月度游客量取自鼓浪屿景区（厦门市）官方月报；2019-08 至 2020-11 是已确认的官方公开数据缺口，处理层保留为缺失，不做插值也不桥接（其余缺失月同样保留为 null）。原始系列覆盖 2015-08 至 2026-04，景区面板 106 行月度，其中 17 个月为公开确认的官方缺口。已解析的官方页中并存多个 `metric_variant`（"上岛游客总量" / "景点接待量" 等），主分析使用"上岛游客总量"口径。气候取自 NASA POWER 网格/再分析日度数据（坐标源自 UNESCO 世界遗产中心 Kulangsu 地图），按月聚合为月均值与月极端日数；该数据源并非本地自动气象站。NASA POWER 日度序列中 3 个 TAVG/PRCP 缺日保留为缺失。月度极端日数（hot 30/35 °C、cold 0 °C、heavy rain 10/25/50 mm）由 NASA POWER 日度层派生，因当前材料不含独立的鼓浪屿极端天气处理层。
