# Chapter 3: Data and System Design

Status: control file
Updated: 2026-06-30

This file is the canonical Chapter 3 control file. The baseline estimator and input block are specified in `design/ms-gam-core.md`. This file documents the observation structure on which the baseline estimator operates.

## 1. Design position

Chapter 3 specifies how the empirical data enter the regime-dependent functional system. It is the observational foundation for the inference, comparison, and forecasting chapters that follow.

The chapter is not a data description. It defines the sampling structure of the regime-dependent functional system: the estimation unit, the temporal frequencies, the climate observation space, and the role of external destinations.

## 2. Estimation unit

This paper is a separate destination-level design, not a pooled panel.

The destination set is:

R = R^{domestic}_1 ∪ R^{domestic}_2 ∪ R^{external}

where:

- R^{domestic}_1: Hong Kong, Macau, Taiwan
- R^{domestic}_2: West Lake, Jiuzhaigou, Gulangyu
- R^{external}: Singapore, Hawaii

Each destination r ∈ R is estimated separately. The baseline MS-GAM estimator specified in `design/ms-gam-core.md` is applied independently to each destination. No shared parameters are estimated across destinations under the baseline specification.

## 3. Temporal sampling structure

Two observation modes are used.

### 3.1 Monthly core

The primary frequency is monthly. The observation grid is:

T_r = {t : t is a monthly observation index for destination r}

The monthly series is the input to the baseline MS-GAM estimator. The current-period monthly block X_t = (T_t, R_t, E_t, m_t, t) enters the baseline observation equation.

### 3.2 Daily support

Daily observations are available for a subset of destinations. As of the current data snapshot, this subset is:

- Macao: daily-to-monthly climate aggregation checks and monthly consistency of extreme indicators.
- Jiuzhaigou: daily flow aggregation checks, daily climate consistency, and month-completeness sensitivity.

Daily data are used to:

- construct the monthly extreme-weather exposure E_t;
- perform aggregation checks and robustness tests.

Daily data are not the input to the baseline MS-GAM estimator. The aggregation-invariance test in Chapter 7 (`design/robustness.md` §5) consumes the same destination list.

### 3.3 Asynchronous structure

Tourism and climate data are asynchronous across destinations. The observation grids T_r for different destinations are not required to be aligned, and the date ranges of available monthly tourism series differ across destinations.

This is not a panel structure with shared time indices. It is a set of separately sampled destination-level time series.

## 4. Climate observation space

The full current-period monthly input block entering the MS-GAM baseline observation equation is the 5-tuple:

X_t = (T_t, R_t, E_t, m_t, t)

where:

- T_t: monthly temperature state, aggregated from daily observations;
- R_t: monthly rainfall state, aggregated from daily observations;
- E_t: extreme-weather exposure at the monthly frequency, constructed from daily observations;
- m_t: calendar month;
- t: time index for the long-run trend.

The climate coordinates of X_t form the climate-state sub-vector:

climate state = (T_t, R_t, E_t)

The 3-tuple "climate state" is **not** a separate object from the 5-tuple X_t; it is the climate sub-vector obtained by stripping the calendar and trend components. The full 5-tuple X_t is what enters the observation equation (`design/ms-gam-core.md` §2). Lagged climate inputs do not enter the baseline.

## 5. Aggregation discipline

Monthly aggregation is the standard projection from daily to monthly:

{x_{r,d}}_{d ∈ month} → x_{r,t}

The aggregation operator preserves:

- mean
- variance
- extreme counts
- frequency summary statistics

Monthly aggregation is not a lossless compression. It is a structured projection that defines the monthly summary statistics. The exact aggregation rule is fixed in the empirical notebooks.

## 6. Daily-monthly bridge

For destinations with daily observations, the daily data are used to construct the monthly E_t and to perform aggregation invariance checks. The aggregation invariance check is a robustness test in Chapter 7, not a baseline estimation step.

## 7. Extreme variable construction

The extreme indicator E_t is constructed from daily observations. Candidate forms include:

- E_t^{bin}: binary event exposure
- E_t^{int}: continuous tail intensity
- E_t^{comp}: compound extreme exposure

The destination-specific threshold τ used to define E_t is fixed in the empirical notebooks. The same threshold is used in `design/ms-gam-core.md` for the extreme term and in `design/prediction.md` for the tail / extreme subset.

## 8. External destinations

The external destinations are Singapore and Hawaii.

Their current role is external validation and comparative extension. They are estimated separately using the same baseline MS-GAM specification. They are not currently part of a transfer learning design, and the baseline does not include cross-destination parameter sharing.

If a future specification introduces a transfer or pooling design that includes these destinations, that specification is an extension and must be presented as such.

## 9. Heterogeneity

Different destinations have different climate-response function families. The paper does not assume f_r = f_s for r ≠ s. The heterogeneity between destination-level systems is the empirical object of Chapter 5, not a nuisance to be pooled away.

## 10. Chapter 3 closure

The observed data are asynchronous, multi-frequency samples drawn from latent destination-specific climate-response systems. The monthly series is the input to the baseline MS-GAM estimator. Daily data support the construction of E_t and the aggregation invariance check. External destinations serve as external validation under the current baseline. The heterogeneity between destination-level systems is preserved by the separate destination-level estimation design.
