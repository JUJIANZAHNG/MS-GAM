# MS-GAM Baseline Estimator: Canonical Specification

Status: canonical control file
Updated: 2026-06-30

This file is the canonical specification of the current baseline MS-GAM estimator. Other design files must reference this file and must not introduce a conflicting version of the state process, observation equation, or input block.

Important convention:

> All formulas are written for one destination. Different destinations are estimated separately, not as a pooled panel.

## 0. Notation glossary (canonical)

Other design files should defer to this glossary. When a file uses a symbol in a way that disagrees with this section, this section is the source of truth.

| Symbol | Meaning | Notes |
|---|---|---|
| r | destination index | the paper uses eight destinations; see `design/data.md` §2 |
| t | monthly time index | the baseline is monthly; daily data are support only |
| K (or K_r) | regime count | under the per-destination convention stated at the top of this file, K is the regime count for the destination r being estimated. Chapter-level files (e.g., `design/method.md`, `design/functional-comparision.md`, `design/outline.md`) write K_r to emphasize that different destinations may have different K. Both forms denote the same object. |
| S_t ∈ {1, ..., K} | latent regime at time t | first-order homogeneous Markov chain |
| X_t = (T_t, R_t, E_t, m_t, t) | current-period monthly input block | the 5-tuple. This is what enters the observation equation. Lagged inputs are not in the block. |
| climate state (T_t, R_t, E_t) | the 3-tuple climate sub-vector of X_t | not a separate object from X_t; just the climate coordinates. The calendar and trend components (m_t, t) are stripped off. |
| f_{r,k}(X_t) or F^{(k)}(X_t) | the regime-specific response function | the scalar observation-equation function for destination r, regime k. Under the per-destination convention, F^{(k)} drops the r subscript. |
| F_r = {f_{r,1}, ..., f_{r,K}} | the family of regime-specific response functions for destination r | a set, not a tuple. Used in the conceptual chapter. |
| \mathcal{F}_r^{(k)} = (f_{r,k}^{temp}, f_{r,k}^{rain}, te_{r,k}, f_{r,k}^{ext}) | the **tuple** of four sub-functions for destination r, regime k | the Chapter 5 geometry object. Calligraphic F is reserved for this tuple, not for the scalar response. |
| \Phi_r^{(k)} = (C_{r,k}^{temp}, C_{r,k}^{rain}, G_{r,k}^{int}, A_{r,k}^{ext}) | the geometric summary of \mathcal{F}_r^{(k)} | the Chapter 5 comparison object. Distinct from \mathcal{F}_r^{(k)}. |
| te_{r,k}(T_t, R_t) | regime-specific temperature-rainfall interaction surface | canonical name. Some chapter-level files write f^{int} for the same object; the canonical form is te_{r,k}. The two names are interchangeable; the canonical form is te. |
| f_{r,k}^{ext}(E_t) | regime-specific explicit extreme-weather term | conceptually distinct from te_{r,k}; do not conflate. See §4 below. |
| Π_t = (π_{t,1}, ..., π_{t,K}) | filtered regime probabilities at t | the forecast-safe structural input. |
| Z_t = Ψ(Π_t) = (Π_t, H_t^Π, Ŝ_t) | forecast-side structural block | the Chapter 6 input. \Phi is **not** a Z input. |
| \hat{P} | estimated transition matrix | K × K, time-invariant under the baseline. |

## 1. State process

The latent tourism-demand regime is:

S_t ∈ {1, ..., K}

The current baseline state process is a first-order homogeneous Markov chain:

P(S_t = j | S_{t-1} = i) = p_{ij}

with transition matrix:

P = (p_{ij})_{K×K}

The transition matrix P is time-invariant under the baseline specification. The extreme term E_t and other climate covariates do not enter the transition equation under the baseline specification. A time-varying transition equation in which E_t enters as a covariate is an extension, not the current baseline.

## 2. Observation equation

For each regime k, define the regime-specific smooth response function:

F^{(k)}(X_t) = s_k^{trend}(t) + s_k^{season}(m_t) + f_k^{temp}(T_t) + f_k^{rain}(R_t) + te_k(T_t, R_t) + f_k^{ext}(E_t)

where:

- t: time index for long-run trend
- m_t: calendar month
- T_t: monthly temperature state
- R_t: monthly rainfall state
- E_t: extreme-weather exposure
- te_k(T_t, R_t): regime-specific temperature-rainfall interaction surface

The observation equation is:

y_t | S_t = k ~ N(F^{(k)}(X_t), σ_k^2)

Equivalently:

y_t = Σ_{k=1}^{K} 1(S_t = k) F^{(k)}(X_t) + ε_t,   ε_t | S_t = k ~ N(0, σ_k^2)

with y_t = log(Y_t + 1).

## 3. Input block discipline

The baseline observation equation uses the current-period monthly block only:

X_t = (T_t, R_t, E_t, m_t, t)

Lagged climate inputs (T_{t-ℓ}, R_{t-ℓ}, E_{t-ℓ} for ℓ ≥ 1) do not enter the baseline estimator. Lagged information is the responsibility of the Chapter 6 forecasting layer, not the baseline MS-GAM estimator.

The climate-memory operator M_L(X_t) defined in Chapter 2 is a conceptual-layer object. It does not enter the current baseline likelihood. A distributed-lag variant of the observation equation, if studied, is an extension or robustness candidate and not the current baseline.

## 4. Extreme term vs. interaction surface

The interaction surface te_k(T_t, R_t) and the extreme term f_k^{ext}(E_t) carry distinct conceptual loads:

- te_k(T_t, R_t) captures the regime-specific continuous interaction between temperature and rainfall at the regular scale.
- f_k^{ext}(E_t) captures explicit extreme-weather exposure, constructed at the monthly frequency from daily observations.

These two terms must not be conflated. The interaction surface is not a substitute for the extreme term, and the extreme term is not a substitute for the interaction surface.

## 5. Transition equation policy

Under the current baseline:

- the transition matrix P is homogeneous in t;
- E_t and other climate covariates do not enter the transition equation;
- regime switching is driven by the inferred state dynamics, not by an explicit covariate-dependent transition specification.

If a future specification allows P_t to depend on E_t or on a regime-specific covariate, that specification is an extension. It must be presented as an extension and must not be conflated with the baseline.

## 6. Role of this file

This file is the canonical baseline specification. Other design files (function-generate.md, framework-ms-gam.md, prediction.md, data.md) must reference this file as the canonical specification of the state process and observation equation. Divergent specifications in other files are not part of the current baseline.
