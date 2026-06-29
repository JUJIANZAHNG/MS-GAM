# Taiwan

## 1. Tourism data

- raw source: Taiwan Tourism Administration cumulative monthly XLSX
- raw frequency: cumulative monthly; monthly recovered by differencing Grand Total row
- processed file: data_processed/tourism/taiwan_visitor_arrivals_month.csv
- time range: 2008-01 to 2026-04

## 2. Climate data

- raw source: Taiwan CWA CODiS StationData
- raw frequency: monthly + daily
- processed files:
  * monthly: taiwan_cwa_monthly_weather_station_panel.csv + taiwan_cwa_monthly_weather_national_summary.csv (6-station simple mean: Taipei, Taichung, Tainan, Hualien, Penghu, Alishan; not population-weighted)
  * daily (current default main): taiwan_cwa_daily_weather_station_panel.csv
  * daily (current default summary): taiwan_cwa_daily_weather_station_summary.csv
  * daily (current default national summary): taiwan_cwa_daily_weather_national_summary.csv
  * daily (version archive copy): taiwan_cwa_daily_weather_station_panel_v3.csv
  * daily (retired 6-station): taiwan_cwa_daily_weather_station_panel_legacy6.csv
- time ranges: monthly panel 1896-08 onward; current default daily panel 2008-01-01 to 2026-06-27 (204,854 rows; 36 in-panel stations)
- KEY POINT: the current default daily files are the unsuffixed names (`taiwan_cwa_daily_weather_station_panel.csv`, `taiwan_cwa_daily_weather_station_summary.csv`, `taiwan_cwa_daily_weather_national_summary.csv`); the `_v3` file is kept only as a version archive copy; the `_legacy6` file is the retired 6-station panel.

## 3. Extreme-weather data

- raw source: Taiwan CWA Typhoon Database (TDB)
- raw frequency: event-level
- processed files: taiwan_tdb_warning_typhoon_events.csv (event-level) + taiwan_tdb_warning_typhoon_monthly_exposure.csv (monthly exposure) + taiwan_tdb_warning_typhoon_year_summary.csv (yearly summary)
- time range: events 1958-07 onward; monthly exposure spans the tourism window

## 4. Processing steps

- Monthly tourism: input Taiwan Tourism Administration cumulative monthly XLSX exports by nationality -> difference the Grand Total row to recover each month's value -> output taiwan_visitor_arrivals_month.csv. Script: fetch_taiwan_arrivals.py.
- Monthly climate: input CWA CODiS StationData JSON per station per month -> build station panel -> compute national simple mean across six curated official CWA stations (Taipei, Taichung, Tainan, Hualien, Penghu, Alishan), no population weighting -> output taiwan_cwa_monthly_weather_station_panel.csv and taiwan_cwa_monthly_weather_national_summary.csv. Script: fetch_taiwan_climate.py.
- Daily climate (current default): input CWA CODiS daily JSON for 43 first-order stations -> build the 204,854-row daily station panel (with 16 skeleton rows added for in-life days that were missing as rows: 永康 2008-07-21, 田中 2020-07-13..14, 玉山 2021-05-30..06-06, 蘇澳 2023-04-01, 高雄 predecessor 2022-01-24) -> apply v3 imputation -> output taiwan_cwa_daily_weather_station_panel.csv (current default main), with the per-station summary taiwan_cwa_daily_weather_station_summary.csv and the national summary taiwan_cwa_daily_weather_national_summary.csv. Script: fetch_taiwan_climate_daily.py.
  * Temperature (5 vars: mean / max / min / daily range / dew point): linear interpolation within station when both neighbours non-null and gap <= 6 days; longer or single-sided gaps kept null.
  * total_precipitation_mm: never time-interpolated; IDW from in-life same-day neighbours within 50 km with neighbour count n >= 3, weights w_i = 1 / (d_i + 1)^2, weighted rain-day probability p_t, zero-rain constraint (p_t < 0.3 -> fill 0, else positive-only log-IDW over the positive neighbours).
  * v3 zero-rain correction: when n >= 3 and all valid neighbours are exactly 0, fill 0 (rule label PRECIP_IDW_ZERO_RAIN_V3); this fixes the v2 logic error of leaving such cells null.
  * v3 two-neighbour relaxation: when n = 2 with the nearest neighbour <= 30 km and both <= 50 km and both have valid same-day precip, apply the same IDW + zero-rain rule, with rule labels PRECIP_IDW_ZERO_RAIN_N2_V3 (p_t < 0.3) or PRECIP_IDW_LOG_AVG_N2_V3 (positive-rain log-IDW) or KEEP_NULL_NO_POSITIVE_NEIGHBOR_N2_V3 (p_t >= 0.3, no positive neighbour).
  * Structural rows (pre-construction, post-closure, radar-only stations 五分山 466850 and 墾丁雷達站 467790, out-of-life offsets 板橋 2023-01-01..16, 新北 2022-03-17..10-22, 七股 2016-06-02..12) are NEVER filled.
  * Sentinel values (-99.5, -99.7, -99.9, -999.5, -9995.0, -99.95, -9999.5) are collapsed to null by an upstream clean_number step.
- Typhoon monthly exposure: input CWA TDB typhoon warning list event-level -> allocate each event's warning hours to calendar months by exact overlap duration -> aggregate to monthly exposure (event count, total hours, max wind speed, min pressure, max 7/10-km radius) -> output taiwan_tdb_warning_typhoon_monthly_exposure.csv (with the event-level taiwan_tdb_warning_typhoon_events.csv and the yearly taiwan_tdb_warning_typhoon_year_summary.csv as parallel layers). Script: fetch_taiwan_typhoon_warning_events.py.

## 5. Missingness / gap filling / aggregation

- v3 imputation results (from .tmp/taiwan_cwb43_daily_panel_before_after_audit_v3.md §1):
  * v2 total_precipitation_mm null = 10,313; v3 null = 8,246 (delta = 2,067 new fills)
  * v3 zero-rain correction: 977 cells
  * v3 two-neighbour relaxation: 1,090 cells (587 zero-rain + 503 log-IDW)
  * v3 temperature linear interpolation: 553 cells (across gap lengths 1-6)
  * skeleton rows added: 16 (5 in-life days that were missing as rows: 永康 2008-07-21, 田中 2020-07-13..14, 玉山 2021-05-30..06-06, 蘇澳 2023-04-01, 高雄 predecessor 2022-01-24)
  * structural rows (never filled): 131
- v3 caveat (must be stated): v3 nc=2 relaxation fills 7 of 8 玉山 2021-05-30..06-06 days and all 5 玉山 2017-06-03..07 days (6/3 estimated at 571.5 mm; sandwiched between 284.5 mm and 3.0 mm). These are the only v3 fills inside a documented extreme/long-gap day. See audit §9.
- Monthly climate: 6-station simple mean (no weighting).
- Typhoon monthly exposure: warning hours split across calendar months by exact overlap duration.

## 6. Final processed files

| file | role | frequency | start | end | note |
|---|---|---|---|---|---|
| taiwan_visitor_arrivals_month.csv | tourism series (monthly differencing of Grand Total) | monthly | 2008-01 | 2026-04 | official Taiwan Tourism Administration series |
| taiwan_cwa_monthly_weather_station_panel.csv | climate monthly station panel | monthly | 1896-08 | 2026-04 | CWA CODiS 43-station monthly panel |
| taiwan_cwa_monthly_weather_national_summary.csv | climate monthly national summary (6-station simple mean) | monthly | 1896-08 | 2026-04 | Taipei, Taichung, Tainan, Hualien, Penghu, Alishan; not population-weighted |
| taiwan_cwa_monthly_weather_station_summary.csv | climate monthly station summary | monthly | 1896-08 | 2026-04 | per-station monthly summary layer |
| taiwan_cwa_daily_weather_station_panel.csv | climate daily station panel (current default main) | daily | 2008-01-01 | 2026-06-27 | 204,854 rows, 36 in-panel stations, v3 imputation rule applied |
| taiwan_cwa_daily_weather_station_summary.csv | climate daily per-station summary (current default summary) | daily | 2008-01-01 | 2026-06-27 | per-station daily summary layer |
| taiwan_cwa_daily_weather_national_summary.csv | climate daily national summary (current default national summary; 6-station simple mean) | daily | 2008-01-01 | 2026-06-27 | built on current default daily panel |
| taiwan_cwa_daily_weather_station_panel_v3.csv | climate daily station panel (version archive copy of v3 imputation) | daily | 2008-01-01 | 2026-06-27 | version archive copy; same content as the current default main file |
| taiwan_cwa_daily_weather_station_panel_legacy6.csv | climate daily station panel (retired 6-station) | daily | — | — | retired 6-station panel; not the current daily series |
| taiwan_tdb_warning_typhoon_events.csv | extreme-weather event-level (typhoon warning) | event-level | 1958-07 | 2024 | CWA TDB warning list |
| taiwan_tdb_warning_typhoon_monthly_exposure.csv | extreme-weather monthly exposure (typhoon warning) | monthly | 1958-07 | 2024 (event) | warning hours split across calendar months by exact overlap |
| taiwan_tdb_warning_typhoon_year_summary.csv | extreme-weather yearly summary (typhoon warning) | yearly | 1958 | 2024 | yearly roll-up of the TDB warning layer |
| taiwan_tourism_climate_ready_panel.csv | tourism x climate ready panel | monthly | 2008-01 | 2026-04 | merges monthly tourism, monthly national climate, monthly typhoon exposure keyed by month |
| taiwan_extreme_climate_month_demo.csv | demo of tourism + climate + warning join | monthly demo | 2008-01 | 2026-04 | (demo, not the primary analysis file) |

## 7. Paper-ready description

**English.** Taiwan's tourism series is monthly inbound visitor arrivals from the official Taiwan Tourism Administration cumulative monthly export, with the monthly value recovered by differencing the Grand Total row (2008-01 to 2026-04). The climate layer comes from Taiwan CWA CODiS StationData. The monthly climate is a 6-station simple mean of curated official CWA stations (Taipei, Taichung, Tainan, Hualien, Penghu, Alishan; not population-weighted) and starts in 1896-08. The daily climate is the current default 43-station panel covering 2008-01-01 to 2026-06-27 (204,854 rows; 36 in-panel stations), with per-station and national summaries. The v3 imputation rule treats five temperature variables (mean / max / min / daily range / dew point) with within-station linear interpolation when both neighbours are non-null and the gap is at most 6 days, and treats total_precipitation_mm with inverse-distance weighting from in-life same-day neighbours within 50 km (n >= 3, weights w_i = 1/(d_i+1)^2, zero-rain constraint p_t < 0.3 -> 0, else positive-only log-IDW). Two v3 override hooks extend the rule: a zero-rain correction (n >= 3 and all valid neighbours exactly 0 -> fill 0) and a two-neighbour relaxation (n = 2 with the nearest neighbour <= 30 km and both <= 50 km, applying the same IDW + zero-rain rule). Structural rows (pre-construction, post-closure, radar-only, out-of-life offsets) are never filled. The extreme-weather layer is the CWA Typhoon Database (TDB); event-level typhoon warnings are allocated to calendar months by exact overlap duration and aggregated to monthly exposure metrics (event count, total hours, max wind speed, min pressure, max 7/10-km radius).

**中文。** 台湾旅游序列为官方观光统计数据库的月度入境游客总量，由累计月报 Grand Total 差分恢复每月值（2008-01 至 2026-04）。气候层来自中央气象署 CODiS 站月/日度资料：月度气候为六个官方站（台北、台中、台南、花莲、澎湖、阿里山）的简单平均（非人口加权），起于 1896-08；日度气候为当前默认 43 站面板，覆盖 2008-01-01 至 2026-06-27（204,854 行，36 个 in-panel 站），并附站点级与全国摘要。v3 填补规则：5 个温度变量（mean / max / min / daily range / dew point）按同测站、两端非缺失、缺口 ≤ 6 日做线性内插；total_precipitation_mm 不做时间方向内插，以同日、生命期内、距离 ≤ 50 公里的邻站（n ≥ 3，权重 w_i = 1/(d_i+1)^2，零雨日约束 p_t < 0.3 → 0，否则正样本对数域 IDW）做反距离加权。v3 增加两个 override hook：①零雨日修正（n ≥ 3 且所有有效邻居皆恰为 0 → 填 0）；②两邻站放宽（n = 2、最近 ≤ 30 公里且两站皆 ≤ 50 公里，应用同一 IDW + 零雨日规则）。结构性行（建站前、闭站后、雷达专用站、超出生命期偏移）永不填补。极端天气层取自中央气象署台风资料库（TDB），事件级台风警报按时段精确切分到日历月，聚合成月度暴露指标（事件数、累计小时、最大风速、最低气压、7/10 公里暴风半径）。
