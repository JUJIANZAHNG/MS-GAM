# Macao

## 1. Tourism data

- **Raw source institution:** DSEC (Macao Statistics and Census Service / 澳門統計暨普查局).
- **Raw carrier / format:** JSON via the official Tourism API endpoint `https://www.dsec.gov.mo/TourismApi/GetHomecodeData`; cross-checked by DSEC PDF monthly bulletin.
- **Raw frequency:** daily (primary, via API) + monthly (cross-check, via PDF).
- **Processed files:**
  - `data_processed/tourism/macao_arrivals_day.csv` — daily series (primary line).
  - `data_processed/tourism/macao_arrivals_month_long.csv` — monthly series parsed from DSEC PDF bulletin (cross-check).
- **Time range:** daily 2017-01-01 to 2026-04-30; monthly window varies by PDF coverage.
- **Key fields:** `date`, `arrivals`, `n_homecodes`, `source`, `frequency` (see `docs/hk-macao-taiwan-data-dictionary.md` §2.2).
- **Script:** `scripts/data_fetch/fetch_macao_tourism_arrivals.py` (daily API line) and `scripts/data_fetch/fetch_macao_arrivals_monthly_pdf.py` (monthly PDF cross-check).
- **KEY POINT — daily window offset:** daily tourism starts 2017-01-01; daily climate (`macao_review_yesterday_daily_weather.csv`) starts 2019-01-01. The daily tourism window therefore contains an additional ~2 years that the daily climate line does not cover. Monthly climate starts 2017-01 and is therefore aligned with the daily tourism start; this 2-year offset must be stated explicitly in §5 and in any paper-ready description.

## 2. Climate data

- **Raw source institution:** Macao SMG (Serviços Meteorológicos e Geofísicos / 地球物理暨氣象局).
- **Raw carrier / format:** monthly PDF (`https://cms.smg.gov.mo/uploads/sync/pdf/CLI/CLI_Month_report/c_resumo_YYYYMM.pdf`, Portuguese original) + daily PDF (`https://cms.smg.gov.mo/uploads/sync/pdf/ReviewYesterday/ReviewYesterday_YYYYMMDD.pdf`).
- **Raw frequency:** monthly (CLI summary) + daily (ReviewYesterday).
- **Processed files:**
  - `data_processed/climate/macao_monthly_weather_summary.csv` — monthly climate (primary monthly line); 2017-01 to 2026-04.
  - `data_processed/climate/macao_review_yesterday_daily_weather.csv` — daily observations; 2019-01-01 to 2026-04-30 (2,677 rows after the §5 bridge).
- **Distinction (do not merge):** the monthly CLI PDF line and the daily ReviewYesterday PDF line are two different series with different start dates and different variable sets. They are NOT a single series, are NOT cross-aggregated, and live in two different files. Treat the two files as two separate climate layers; if a monthly ready panel is needed, prefer the monthly CLI file (aligned with daily tourism from 2017-01) and only use the daily file for daily-frequency analyses.
- **Script:** `scripts/data_fetch/fetch_macao_climate.py` (handles both monthly CLI PDF and daily ReviewYesterday PDF, plus the §5 gap-fill log).
- **Key fields (daily):** `date`, `max_temperature_c`, `min_temperature_c`, `mean_wind_speed_kmh`, `total_rainfall_mm`, `max_hourly_rainfall_mm`, `max_gust_kmh`, `weather_phenomena_zh`/`_pt`, plus `_missing_flag` and `_trace_flag` columns (see `docs/hk-macao-taiwan-data-dictionary.md` §2.4).
- **Key fields (monthly):** `month`, `mean_temperature_c`, `mean_max_temperature_c`, `mean_min_temperature_c`, `total_rainfall_mm`, `cumulative_rainfall_mm`, `max_daily_rainfall_mm`, `mean_relative_humidity_pct`, `evaporation_mm`, `sunshine_hours`, `mean_wind_speed_kmh`, `prevailing_wind_direction` (see `docs/hk-macao-taiwan-data-dictionary.md` §2.3).

## 3. Extreme-weather data

Macao has **four separate SMG warning families**. Each family is held in its own file trio (event-level + day-level + monthly-exposure; some families also have a separate `*_signal_records.csv` / `*_events.csv`). The four families are **not merged into a single extreme-weather index**.

### 3.1 Rainstorm (暴雨警告)

- **Source page:** `https://www.smg.gov.mo/zh/subpage/355/report/rainstorm-history`; PDF family `https://cms.smg.gov.mo/uploads/sync/pdf/rs/rsYYYY/c_rainstorm_YYYYMMDD.pdf`.
- **Frequency in raw:** event-level (one PDF per event); processed into event/day/monthly.
- **Processed files:**
  - `macao_smg_rainstorm_warning_events.csv` — event-level (yellow / red / black signal sequence per event).
  - `macao_smg_rainstorm_warning_day.csv` — daily-level exposure.
  - `macao_smg_rainstorm_warning_monthly_exposure.csv` — monthly exposure (event count, total hours, max signal, colour-specific counts, `has_rainstorm_warning`).
- **Time range:** 2000-04-03 to 2026-06-07.
- **Script:** `scripts/data_fetch/fetch_macao_rainstorm_warnings.py`.
- **Caveat:** covers rainstorm warnings only; one legacy PDF's cancellation time was not parseable so that event's `duration_hours_calc` is left blank.

### 3.2 Tropical cyclone signal (熱帶氣旋信號)

- **Source page:** `https://www.smg.gov.mo/zh/subpage/355/report/typhoon-yearly-report`; PDF family `https://cms.smg.gov.mo/uploads/sync/pdf/tc/YYYY/c_*.pdf`; name mapping `https://cms.smg.gov.mo/uploads/sync/pdf/tc/typhoon_report_mapping.json`.
- **Frequency in raw:** event-level (one PDF per TC report); processed into event/day/monthly.
- **Processed files:**
  - `macao_smg_tropical_cyclone_events.csv` — TC report-level events.
  - `macao_smg_tropical_cyclone_signal_records.csv` — signal-level records (each individual signal raise/lower).
  - `macao_smg_tropical_cyclone_warning_day.csv` — daily-level exposure.
  - `macao_smg_tropical_cyclone_monthly_exposure.csv` — monthly exposure (event count, total hours, max signal number, 8/9/10-号風球 counts, `has_tc_warning`).
- **Time range:** 2000-07-06 to 2025-11-10.
- **Script:** `scripts/data_fetch/fetch_macao_tropical_cyclone_warnings.py`.
- **Caveat:** only the TC signal timeline is the main variable; other tables inside the TC report PDFs are NOT mixed into the main variable.

### 3.3 Storm surge (風暴潮警告)

- **Source page:** `https://www.smg.gov.mo/zh/subpage/355/page/39`; official API `https://cms.smg.gov.mo/zh_TW/api/sitecontent/39`.
- **Frequency in raw:** event-level (advisory / formal-warning split); processed into event/day/monthly.
- **Processed files:**
  - `macao_smg_storm_surge_warning_events.csv` — event-level (with `has_advisory` / `has_formal_warning` flags).
  - `macao_smg_storm_surge_signal_records.csv` — signal-level records.
  - `macao_smg_storm_surge_warning_day.csv` — daily-level exposure.
  - `macao_smg_storm_surge_monthly_exposure.csv` — monthly exposure (event count, formal-warning hours, advisory hours, max signal, yellow+/red+ counts, `has_storm_surge_exposure`).
- **Time range:** 2009-08-05 to 2025-10-04.
- **Script:** `scripts/data_fetch/fetch_macao_storm_surge_warnings.py`.
- **Caveat:** "風暴潮戒備訊息" (advisory) and formal warning are tracked separately; the monthly exposure file splits hours across calendar months by **actual exposure hours**, not by event start month. A separate `sitecontent/38` historical water-level table is an intensity supplement only — it is NOT a continuous event series.

### 3.4 Thunderstorm (雷暴)

- **Status:** thunderstorm is **currently NOT a continuous series**. The raw warning record is institutionally published as a regime description, not as a structured historical event list, so no event-level or monthly-exposure file is built from a structured source.
- **Processed files (current scope):**
  - `macao_smg_thunderstorm_warning_events.csv` — placeholder / future-state file; not a populated event series.
  - `macao_smg_thunderstorm_warning_monthly_exposure.csv` — placeholder / future-state file; not a populated monthly exposure series.
- **Script:** `scripts/data_fetch/fetch_macao_thunderstorm_warnings.py` (does not yet produce a continuous event series).
- **Flag:** this family must be reported in any paper-ready description as "not yet a continuous series" — do NOT treat it as equivalent to the other three families in coverage or readiness.

## 4. Processing steps

### 4.1 Tourism — daily

- **Input:** DSEC Tourism API `GetHomecodeData` (JSON) — per-homecode daily visitor records.
- **Intermediate processing:** per-homecode records are grouped by date and summed to a total-daily-arrivals field; the JSON is flattened into a tabular daily series; `n_homecodes` is recorded.
- **Output:** `data_processed/tourism/macao_arrivals_day.csv`.
- **Script:** `scripts/data_fetch/fetch_macao_tourism_arrivals.py`.

### 4.2 Tourism — monthly (PDF)

- **Input:** DSEC PDF monthly bulletin.
- **Intermediate processing:** PDF parsed into a monthly arrivals series; aligned with the daily series for cross-check purposes.
- **Output:** `data_processed/tourism/macao_arrivals_month_long.csv`.
- **Script:** `scripts/data_fetch/fetch_macao_arrivals_monthly_pdf.py`.

### 4.3 Climate — monthly

- **Input:** SMG monthly CLI summary PDF (Portuguese original, one per month).
- **Intermediate processing:** PDF parsed into English-labelled numeric fields (`mean_temperature_c`, `mean_max_temperature_c`, `mean_min_temperature_c`, `total_rainfall_mm`, `cumulative_rainfall_mm`, `max_daily_rainfall_mm`, `mean_relative_humidity_pct`, `evaporation_mm`, `sunshine_hours`, `mean_wind_speed_kmh`, `prevailing_wind_direction`); trace-rainfall values are flagged via `total_rainfall_trace_flag` / `max_daily_rainfall_trace_flag` while the numeric column itself keeps the SMG-published value.
- **Output:** `data_processed/climate/macao_monthly_weather_summary.csv`.
- **Script:** `scripts/data_fetch/fetch_macao_climate.py` (monthly branch).

### 4.4 Climate — daily

- **Input:** SMG ReviewYesterday daily PDF (one per day).
- **Intermediate processing:** PDF parsed into a daily observation series; for the single 2019-02-26 PDF-404 day, a processed-layer bridge row is added (see §5).
- **Output:** `data_processed/climate/macao_review_yesterday_daily_weather.csv` + `data_processed/climate/macao_review_yesterday_gap_fill_log.csv`.
- **Script:** `scripts/data_fetch/fetch_macao_climate.py` (daily branch; same script writes the gap-fill log).

### 4.5 Rainstorm warnings

- **Input:** SMG rainstorm PDF family `c_rainstorm_YYYYMMDD.pdf` (one per event).
- **Intermediate processing:** PDF parsed for start/end times and signal sequence (yellow → red → black); event-level table built; events exploded to daily and aggregated to monthly exposure.
- **Output:** `macao_smg_rainstorm_warning_events.csv` + `macao_smg_rainstorm_warning_day.csv` + `macao_smg_rainstorm_warning_monthly_exposure.csv`.
- **Script:** `scripts/data_fetch/fetch_macao_rainstorm_warnings.py`.

### 4.6 Tropical cyclone warnings

- **Input:** SMG TC report PDF family (`c_*.pdf`) under `pdf/tc/YYYY/`, plus the `typhoon_report_mapping.json` name table.
- **Intermediate processing:** TC report PDF parsed for signal timeline and TC metadata; signal-level records table and event-level table built; events exploded to daily and aggregated to monthly exposure.
- **Output:** `macao_smg_tropical_cyclone_events.csv` + `macao_smg_tropical_cyclone_signal_records.csv` + `macao_smg_tropical_cyclone_warning_day.csv` + `macao_smg_tropical_cyclone_monthly_exposure.csv`.
- **Script:** `scripts/data_fetch/fetch_macao_tropical_cyclone_warnings.py`.

### 4.7 Storm surge warnings

- **Input:** SMG sitecontent/39 JSON for the storm surge history; "戒備訊息" advisory and formal warning tracked separately.
- **Intermediate processing:** JSON parsed into advisory / formal-warning timelines; event-level table built; events exploded to daily and aggregated to monthly exposure; monthly hours are allocated by **actual exposure hours**, not by event start month.
- **Output:** `macao_smg_storm_surge_warning_events.csv` + `macao_smg_storm_surge_signal_records.csv` + `macao_smg_storm_surge_warning_day.csv` + `macao_smg_storm_surge_monthly_exposure.csv`.
- **Script:** `scripts/data_fetch/fetch_macao_storm_surge_warnings.py`.

### 4.8 Thunderstorm warnings

- **Input:** SMG thunderstorm public materials (currently descriptive, not a structured historical event list).
- **Intermediate processing:** none stable yet — see §3.4 flag.
- **Output (placeholder, not populated):** `macao_smg_thunderstorm_warning_events.csv` + `macao_smg_thunderstorm_warning_monthly_exposure.csv`.
- **Script:** `scripts/data_fetch/fetch_macao_thunderstorm_warnings.py` (does not yet emit a continuous event series).

## 5. Missingness / gap filling / aggregation

### 5.1 Daily climate (ReviewYesterday) — THE ONLY gap fill

- The raw `ReviewYesterday` PDF series has exactly **one** official missing day: `2019-02-26` (PDF returns 404). The raw missing-date manifest is preserved in `data_raw/macao_climate/macao_review_yesterday_missing_dates.csv`.
- In the **processed** layer (`macao_review_yesterday_daily_weather.csv`), this single day is bridged by **adjacent-day linear interpolation** of the continuous numeric weather fields: `max_temperature_c`, `min_temperature_c`, `mean_wind_speed_kmh`, `max_relative_humidity_pct`, `min_relative_humidity_pct`, `max_gust_kmh`, `total_rainfall_mm`, `evaporation_mm`, `sunshine_hours`. The bridge row is identified by `source_family = review_yesterday_interpolated`.
- The following fields are **not** interpolated and remain missing in the bridge row: `max_hourly_rainfall_mm`, `prevailing_wind_direction`, `weather_phenomena_zh`, `weather_phenomena_pt`.
- The bridge is recorded in `macao_review_yesterday_gap_fill_log.csv` (one row for `2019-02-26`).
- The processed daily file therefore has 2,677 rows; the raw missing-date manifest still has 1 row.
- Decision provenance: `.planning/2026-06-22-macao-climate-daily-gap-fill/session.md`.
- **This is the only gap fill in the Macao processing pipeline.** Do not describe Macao as having an interpolation step; describe it as "an isolated one-day bridge on 2019-02-26".

### 5.2 Four warning families — no fill

- Event absence in any of the four SMG warning families is **meaningful**: a month with no rainstorm warning is genuinely a month with no rainstorm warning, and is **not** filled.
- Cancellation-time parse failures: in the rainstorm event file, one legacy PDF's cancellation time was not parseable, so that event's `duration_hours_calc` is left blank — this is a missing-value-in-event-record, not a gap-fill.
- Storm-surge monthly exposure hours are allocated by **actual exposure hours** across calendar months; this is an allocation rule, not a fill.

### 5.3 Tourism — no fill

- Daily tourism (`macao_arrivals_day.csv`) is not filled; any missing day is preserved as missing in the processed series.
- Monthly tourism (`macao_arrivals_month_long.csv`) is a PDF-derived cross-check line; the daily API line is the primary.

### 5.4 Monthly climate — no fill

- The monthly CLI line (`macao_monthly_weather_summary.csv`) is not filled; trace-rainfall values are kept as the SMG-published numeric value with `*_trace_flag` retained.

### 5.5 2-year offset — daily tourism vs daily climate

- Daily tourism starts **2017-01-01**; daily climate starts **2019-01-01**. The daily tourism window therefore contains an additional ~2 years that the daily climate line does not cover.
- Monthly climate starts **2017-01** and is therefore aligned with the daily tourism start; a monthly-frequency analysis can use the full daily-tourism window.
- This boundary must be stated explicitly in any paper-ready description, and any daily-frequency regression must either restrict the analysis window to 2019-01-01 onwards, or handle the 2017-2018 window as "tourism-without-daily-climate" (the official review above is that current materials treat the 2-year window as a structural boundary, not a missing-data issue).

### 5.6 Aggregations

- **Daily → monthly (climate):** monthly total rainfall is the sum of daily `total_rainfall_mm`; monthly mean temperature is the mean of daily mean-temperature proxies; these are derived in the monthly CLI line directly from the PDF (not by re-aggregating the daily file).
- **Event → daily (warnings):** each warning event's start/end times are exploded to a daily series; the daily file records, for each calendar day, whether that family was active and at what level.
- **Event → monthly exposure (warnings):** for rainstorm, tropical cyclone, and storm surge, monthly exposure metrics (event count, total hours, max signal level, signal-specific counts) are aggregated by calendar month. For storm surge specifically, monthly hours are split across calendar months by **actual exposure hours**, not by event start month.
- **Daily → monthly (tourism):** the monthly tourism file is parsed from the PDF bulletin; it is treated as a cross-check rather than a re-aggregation of the daily series, but the two are aligned for cross-validation.

## 6. Final processed files

| file | role | frequency | start | end | note |
|---|---|---|---|---|---|
| `macao_arrivals_day.csv` | tourism line, primary | daily | 2017-01-01 | 2026-04-30 | DSEC API daily arrivals; no fill |
| `macao_arrivals_month_long.csv` | tourism line, cross-check | monthly | varies | varies | DSEC PDF bulletin; cross-check vs daily |
| `macao_monthly_weather_summary.csv` | climate line, primary monthly | monthly | 2017-01 | 2026-04 | SMG monthly CLI PDF |
| `macao_review_yesterday_daily_weather.csv` | climate line, daily | daily | 2019-01-01 | 2026-04-30 | SMG daily PDF; one-day bridge for 2019-02-26 |
| `macao_review_yesterday_gap_fill_log.csv` | climate daily, gap-fill log | one row | 2019-02-26 | 2019-02-26 | only gap fill in the Macao pipeline |
| `macao_smg_rainstorm_warning_events.csv` | warning family 1, events | event-level | 2000-04-03 | 2026-06-07 | rainstorm PDF family |
| `macao_smg_rainstorm_warning_day.csv` | warning family 1, day | daily | 2000-04-03 | 2026-06-07 | exploded from events |
| `macao_smg_rainstorm_warning_monthly_exposure.csv` | warning family 1, monthly exposure | monthly | 2000-04 | 2026-06 | aggregate of events into months |
| `macao_smg_tropical_cyclone_events.csv` | warning family 2, events | event-level | 2000-07-06 | 2025-11-10 | TC report PDFs |
| `macao_smg_tropical_cyclone_signal_records.csv` | warning family 2, signal records | event-level (signal) | 2000-07-06 | 2025-11-10 | per-signal raise/lower record |
| `macao_smg_tropical_cyclone_warning_day.csv` | warning family 2, day | daily | 2000-07-06 | 2025-11-10 | exploded from events |
| `macao_smg_tropical_cyclone_monthly_exposure.csv` | warning family 2, monthly exposure | monthly | 2000-07 | 2025-11 | aggregate of events into months |
| `macao_smg_storm_surge_warning_events.csv` | warning family 3, events | event-level | 2009-08-05 | 2025-10-04 | advisory + formal warning split |
| `macao_smg_storm_surge_signal_records.csv` | warning family 3, signal records | event-level (signal) | 2009-08-05 | 2025-10-04 | per-signal raise/lower record |
| `macao_smg_storm_surge_warning_day.csv` | warning family 3, day | daily | 2009-08-05 | 2025-10-04 | exploded from events |
| `macao_smg_storm_surge_monthly_exposure.csv` | warning family 3, monthly exposure | monthly | 2009-08 | 2025-10 | hours split by actual exposure across months |
| `macao_smg_thunderstorm_warning_events.csv` | warning family 4, events | event-level (placeholder) | — | — | not a continuous series yet |
| `macao_smg_thunderstorm_warning_monthly_exposure.csv` | warning family 4, monthly exposure | monthly (placeholder) | — | — | not a continuous series yet |
| `macao_extreme_climate_month_demo.csv` | join demo, monthly | monthly demo | 2017-01 | 2026-04 | (demo, not the primary analysis file) |
| `macao_extreme_climate_day_demo.csv` | join demo, daily | daily demo | 2019-01-01 | 2026-04-30 | (demo, not the primary analysis file) |

## 7. Paper-ready description

**English (≈ 200 words).** Macao daily visitor arrivals are taken from the Macao Statistics and Census Service (DSEC) Tourism API, aggregated from per-homecode records to total daily arrivals, and cover 2017-01-01 to 2026-04-30. A separate monthly arrivals series is parsed from the DSEC PDF bulletin as a cross-check. Macao normal-climate monthly variables (monthly mean temperature, monthly total rainfall) are parsed from the Macao SMG monthly CLI summary PDFs and cover 2017-01 to 2026-04; the daily ReviewYesterday series covers 2019-01-01 to 2026-04-30 and starts two years after the daily tourism series — a structural boundary that any daily-frequency analysis must state explicitly. The single 2019-02-26 PDF-404 in the daily climate is bridged in the processed layer by adjacent-day linear interpolation of continuous numeric fields, while `max_hourly_rainfall_mm`, wind direction, and weather text remain missing; the bridge is logged in `macao_review_yesterday_gap_fill_log.csv` and is the only gap fill in the Macao pipeline. Macao extreme weather is organised as four separate SMG warning families — rainstorm, tropical cyclone signal, storm surge, and thunderstorm — each parsed to event/day/monthly exposure and NOT merged into a single extreme-weather index; thunderstorm is currently not a continuous series.

**中文（≈ 220 字）。** 澳门日度访澳旅客数据取自澳门统计暨普查局（DSEC）旅游 API，按入境口岸（homecode）记录聚合成每日总量，覆盖 2017-01-01 至 2026-04-30；另有从 DSEC 月度公报 PDF 解析的月度序列作为交叉验证层。澳门常态气候变量（月平均气温、月总降水量）从地球物理暨气象局（SMG）月度气候总结 PDF 解析，覆盖 2017-01 至 2026-04；日度 ReviewYesterday 系列覆盖 2019-01-01 至 2026-04-30，比日度旅游晚 2 年，这一结构性边界必须在方法叙述中显式说明。唯一公开缺失日 2019-02-26（PDF 404）在处理层以相邻日线性插值填补，仅作用于连续数值气象量，`max_hourly_rainfall_mm`、风向、天气文本保持缺失，补点记录在 `macao_review_yesterday_gap_fill_log.csv` 中；这是澳门处理流程中唯一的一次补点。澳门极端天气按四条独立 SMG 警告族（暴雨、热带气旋信号、风暴潮、雷暴）分别整理为事件/日/月暴露指标，不合并为单一极端天气指数；雷暴当前尚不构成连续事件序列。
