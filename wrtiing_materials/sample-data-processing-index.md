# Sample Data Processing — Index

> 总索引：汇总 8 个样本的数据处理记录文件，每个样本在主表中列一行。
> 仅做汇总，不展开内容。详细内容见各样本记录文件。
> 索引建立日期：2026-06-29

## 1. 样本数据处理记录汇总表

| sample | tourism frequency | climate frequency | extreme-weather layer | main processed files | record file path |
|---|---|---|---|---|---|
| Hong Kong | monthly (HKTB XLS, 2002-01 → 2026-04) | monthly HKO HQ (1884-01 → 2026-05) + daily HKO AWS station panel (1984-10-02 → 2026-05-31) + HKO TC impact tables (1946 → 2024) | yes (8 HKO warning families: cold / fire / frost / hot / landslip / black-rainfall / tropical-cyclone / thunderstorm) — event-level + monthly aggregation | `hong_kong_visitor_arrivals_month.csv`; `hong_kong_hko_monthly_weather.csv`; `hong_kong_station_daily_weather_panel.csv`; `hong_kong_station_monthly_weather_panel.csv`; `hong_kong_warning_events.csv`; `hong_kong_warning_event_month.csv`; `hong_kong_tc_impact_signal_events.csv`; `hong_kong_tc_impact_rainfall_events.csv`; `hong_kong_tc_impact_other_met_events.csv` | `wrtiing_materials/hong-kong-data-processing-record.md` |
| Macao | daily (DSEC Tourism API, 2017-01-01 → 2026-04-30) + monthly (DSEC PDF bulletin, varies; cross-check only) | monthly SMG (2017-01 → 2026-04) + daily SMG ReviewYesterday (2019-01-01 → 2026-04-30) | yes (4 SMG warning families: rainstorm / tropical cyclone signal / storm surge / thunderstorm) — event-level + daily + monthly exposure; thunderstorm not yet continuous | `macao_arrivals_day.csv`; `macao_monthly_weather_summary.csv`; `macao_review_yesterday_daily_weather.csv`; `macao_smg_rainstorm_warning_*.csv`; `macao_smg_tropical_cyclone_*.csv`; `macao_smg_storm_surge_*.csv`; `macao_smg_thunderstorm_warning_*.csv` | `wrtiing_materials/macao-data-processing-record.md` |
| Taiwan | monthly (Taiwan Tourism Administration cumulative XLSX, 2008-01 → 2026-04; monthly recovered by differencing Grand Total) | monthly CWA CODiS 6-station simple mean (1896-08 → 2026-04) + daily CWA CODiS 43-station panel (2008-01-01 → 2026-06-27) | yes (CWA Typhoon Database TDB — event-level + monthly exposure allocated by exact overlap) | `taiwan_visitor_arrivals_month.csv`; `taiwan_cwa_monthly_weather_station_panel.csv`; `taiwan_cwa_monthly_weather_national_summary.csv`; `taiwan_cwa_daily_weather_station_panel.csv` (current default main); `taiwan_cwa_daily_weather_station_summary.csv` (current default summary); `taiwan_cwa_daily_weather_national_summary.csv` (current default national summary); `taiwan_cwa_daily_weather_station_panel_v3.csv` (version archive copy); `taiwan_cwa_daily_weather_station_panel_legacy6.csv` (retired 6-station); `taiwan_tdb_warning_typhoon_events.csv`; `taiwan_tdb_warning_typhoon_monthly_exposure.csv`; `taiwan_tourism_climate_ready_panel.csv` | `wrtiing_materials/taiwan-data-processing-record.md` |
| West Lake | monthly (HTML page, raw 2016-01 → 2026-04; strict analysis 2016-01 → 2025-12 due to incomplete 2026) | daily NASA POWER (gridded/reanalysis at UNESCO West Lake coord, 1981-01-01 → 2026-06-05); alternative unpromoted: Hangzhou Open Data (9 sub-datasets, sub-daily) | no dedicated processed layer; monthly extreme-day counts derived from NASA POWER daily | `westlake_scenic_spot_flow_month.csv`; `westlake_scenic_spot_flow_quarter.csv`; `nasa_power_scenic_daily_weather.csv` (West Lake rows); `nasa_power_scenic_monthly_weather.csv` (West Lake rows); `scenic_tourism_climate_month_panel.csv` (West Lake rows) | `wrtiing_materials/west-lake-data-processing-record.md` |
| Gulangyu | monthly (HTML page, 2015-08 → 2026-04; 2019-08 → 2020-11 is a publicly confirmed official data gap, left as null) | daily NASA POWER (gridded/reanalysis at UNESCO Gulangsu coord, 1981-01-01 → 2026-06-05) | no dedicated processed layer; monthly extreme-day counts derived from NASA POWER daily | `gulangyu_tourism_month.csv`; `nasa_power_scenic_daily_weather.csv` (Gulangyu rows); `nasa_power_scenic_monthly_weather.csv` (Gulangyu rows); `scenic_tourism_climate_month_panel.csv` (Gulangyu rows) | `wrtiing_materials/gulangyu-data-processing-record.md` |
| Jiuzhaigou | daily (official HTML, 2015-06-13 → 2026-06-02) — THREE layers: raw (no fill) / bridged (32 single-day bridge rows outside closure windows) / month_bridged (sum of bridged daily) | daily NASA POWER (gridded/reanalysis at UNESCO Jiuzhaigou coord, 1981-01-01 → 2026-06-05) | no dedicated processed layer; monthly extreme-day counts derived from NASA POWER daily | `jiuzhai_arrivals_day.csv` (raw); `jiuzhai_arrivals_day_bridged.csv`; `jiuzhai_arrivals_day_gap_fill_log.csv`; `jiuzhai_arrivals_month_bridged.csv`; `jiuzhai_arrivals_processing_change_summary.csv`; `docs/jiuzhai_closure_reopen_audit.md`; `nasa_power_scenic_daily_weather.csv` (Jiuzhaigou rows); `nasa_power_scenic_monthly_weather.csv` (Jiuzhaigou rows) | `wrtiing_materials/jiuzhaigou-data-processing-record.md` |
| Singapore | monthly (SingStat Table Builder M550001, 2008-01 → 2026-04) | monthly NEA / data.gov.sg (1982-01 → 2026-05) + daily Changi S24 from weather.gov.sg (2008-01-01 → 2026-05-31) — two parallel layers, NOT bridged | no dedicated processed layer | `singapore_arrivals_month.csv`; `singapore_monthly_weather_summary.csv`; `singapore_changi_daily_weather.csv`; `singapore_tourism_climate_ready_panel.csv` | `wrtiing_materials/singapore-data-processing-record.md` |
| Hawaii | monthly (UHERO/DBEDT TDW API VV101/MM102/DI10, 1989-01 → 2026-04) | monthly NCEI Climate at a Glance statewide time series 51 (1991-01 → 2026-05) + daily NOAA GHCN-Daily multi-station pool (default 7 stations; multiple coverage windows) — two parallel layers, NOT bridged | no dedicated processed layer | `hawaii_arrivals_month.csv`; `hawaii_monthly_weather_summary.csv`; `hawaii_ghcn_daily_weather.csv`; `hawaii_ghcn_common_stations_*.csv` (multi-window); `hawaii_tourism_climate_ready_panel.csv` | `wrtiing_materials/hawaii-data-processing-record.md` |

## 2. 9 个文件清单

1. `wrtiing_materials/hong-kong-data-processing-record.md`
2. `wrtiing_materials/macao-data-processing-record.md`
3. `wrtiing_materials/taiwan-data-processing-record.md`
4. `wrtiing_materials/west-lake-data-processing-record.md`
5. `wrtiing_materials/gulangyu-data-processing-record.md`
6. `wrtiing_materials/jiuzhaigou-data-processing-record.md`
7. `wrtiing_materials/singapore-data-processing-record.md`
8. `wrtiing_materials/hawaii-data-processing-record.md`
9. `wrtiing_materials/sample-data-processing-index.md`（本文件）

## 3. 信息不完整样本

按用户要求"如果某个环节信息不够，就明确写'当前材料不足以确认'"，以下样本仍存在信息缺口：

- **Gulangyu**：处理变更摘要（`scenic_tourism_climate_month_panel_missing_months.csv`）逐月缺口的具体行数未在记录文件中列出。
- **West Lake**：季度文件 `westlake_scenic_spot_flow_quarter.csv` 的精确起止日期未在记录文件中列出（季度记录在处理时只记录"259 rows"，未单独校验起止月份）。Hangzhou Open Data 9 个子数据集的覆盖时段未合并评估。
- **Singapore**：樟宜日度层在 monthly ready panel 中"未聚合"是已确认事实，但 4 个 data.gov.sg 月度 series 的具体 resource_id 列表未在记录文件的表格中逐行列出（已用 `singapore_climate_source_summary.csv` 引用）。
- **Hawaii**：GHCN-Daily 多套覆盖窗口（default 7 / modern32 / relaxed 1989-2025 / relaxed 1998-2023 / relaxed 2000-2024）逐套的精确起止日期未在记录文件中逐套列出；默认 7 站池的具体经纬度与可用时段未在记录文件中列出。
- **Jiuzhaigou**：32 桥接日的逐日 `interpolated_float` 数值未在记录文件的表格中列出（仅引用 `jiuzhai_arrivals_day_gap_fill_log.csv` 与 `docs/jiuzhai_closure_reopen_audit.md`）。
- **Macao**：4 族 warning 中 thunderstorm 至今未形成连续 series 的具体时间范围未在记录文件中明确标出（仅标"not yet a continuous series"）。
- **Hong Kong**：HKO 8 族 warning 各自的精确起止日期未在记录文件中逐族列出（仅标"event-level files"）。
- **Taiwan**：当前默认主日度面板的 36 站具体站号列表（哪些 7 站被排除、为什么）未在记录文件中逐站列出（已用 `.tmp/taiwan_cwb43_daily_panel_before_after_audit_v3.md` 引用）。
