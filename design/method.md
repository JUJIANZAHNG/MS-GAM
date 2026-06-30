# Chapter 2: Conceptual Framework

Status: control file
Updated: 2026-06-30

## 1. Central view

Tourism demand is modeled as a regime-dependent functional system defined for each destination r. At the **conceptual layer**, the climate state entering the system is the 3-tuple:

C_t = (T_t, R_t, E_t)

where T_t is the monthly temperature state, R_t is the monthly rainfall state, and E_t is the extreme-weather exposure constructed at the monthly frequency. This C_t is the conceptual-layer object — it is **not** the empirical X_t input block. The empirical X_t = (T_t, R_t, E_t, m_t, t) is a 5-tuple (calendar and trend are added); see `design/ms-gam-core.md` §0 for the canonical glossary and `design/data.md` §4 for the input-block definition. The conceptual C_t and the climate sub-vector of the empirical X_t share the same three components, but the conceptual layer does not need calendar or trend.

The conceptual equation is:

Y_{r,t} = f_{r, S_{r,t}}(M_L(C_t)) + ε_{r,t}

where M_L(C_t) is the climate-memory operator acting on the climate state, S_{r,t} ∈ {1, ..., K_r} is the latent regime, and ε_{r,t} is the residual.

This equation is the conceptual mother equation of the paper. It is a conceptual statement. It is not the current baseline MS-GAM estimator's likelihood equation. The current baseline estimator is specified in `design/ms-gam-core.md`.

## 2. Climate memory operator

Define:

M_L(C_t) = (C_t, C_{t-1}, ..., C_{t-L})

The operator M_L is a conceptual-layer object. Its primary role is to define the family of inputs available to forecasting models in Chapter 6, where lagged information and forecast-available climate blocks become explicit inputs.

The lagged components of M_L do not enter the current baseline MS-GAM estimator as likelihood inputs. The baseline observation equation uses the current-period monthly block X_t only. If a distributed-lag variant of the MS-GAM observation equation is studied, it is an extension or robustness specification, not the current baseline.

## 3. Regime-specific function family

For destination r, the conceptual object is the family of regime-specific response functions:

F_r = {f_{r,1}, ..., f_{r,K_r}}

Each destination is treated as a dynamic system whose response surface can change across latent structural states. Different destinations are allowed to have different function families: F_r ≠ F_s for r ≠ s. The chapter does not claim that these families share parameters across destinations.

## 4. Decomposition of the response function

At the empirical layer, each regime-specific function decomposes into:

f_{r,k}(X_t) = f_{r,k}^{temp}(T_t) + f_{r,k}^{rain}(R_t) + te_{r,k}(T_t, R_t) + f_{r,k}^{ext}(E_t)

where:

- f^{temp}: nonlinear temperature effect
- f^{rain}: nonlinear rainfall effect
- te: continuous temperature-rainfall interaction surface (canonical name; see `design/ms-gam-core.md` §0)
- f^{ext}: explicit extreme-weather exposure

The interaction surface and the extreme term are conceptually distinct objects and must not be conflated. The interaction surface te_{r,k} carries the continuous temperature-rainfall interaction at the regular scale. The extreme term f^{ext} carries explicit extreme-weather exposure constructed at the monthly frequency.

## 5. Regime process

S_{r,t} ∈ {1, ..., K_r}

Conceptually, regime-specific functions can switch across latent states over time:

f_{r, S_{r,t}} ≠ f_{r, S_{r,t+1}} in general

This switching potential is the theoretical reason the paper adopts an MS-GAM design rather than a single-GAM design. It is a statement about the conceptual layer, not a claim that the current baseline estimator has already identified time-varying transition dynamics.

## 6. Conceptual role of extremes

The extreme-weather term E_t is part of the climate state, but its role extends beyond a generic covariate. Conceptually, extreme conditions can enter the system through three channels:

| Channel | Meaning |
|---|---|
| direct exposure | enter the response function through f_{r,k}^{ext}(E_t) |
| interaction amplification | strengthen or reshape the temperature-rainfall surface |
| state perturbation | make regime switching more likely |

### 6.1 Claim boundary

The current baseline MS-GAM uses a first-order homogeneous Markov transition matrix. The "state perturbation" channel — extreme-driven time-varying transition — is a theoretical motivation and a candidate extension. It is not a channel that the current baseline estimator formally identifies.

The baseline observation equation already includes the direct-exposure channel through f_k^{ext}(E_t) and partially accommodates the interaction-amplification channel through te_k(T_t, R_t). The qualifier "partially" refers to *channel coverage*, not to a partial coefficient: te_k is a full regime-specific component in the observation equation (`design/ms-gam-core.md` §2), but it captures only the *static* continuous T–R interaction. An E_t-modulated interaction surface (e.g., te_k(T_t, R_t, E_t)) that would let extreme events reshape the interaction is not identified under the current baseline. The state-perturbation channel is not represented in the current baseline. Any claim about extreme-driven transition dynamics belongs to a time-varying-transition variant of the model, which is out of scope for the baseline specification in `design/ms-gam-core.md`.

## 7. One-line summary

Tourism demand is modeled as a destination-specific, regime-dependent climate-response system defined over a memory-enhanced climate state, with explicit separation between regular climate effects, interaction structure, and extreme-weather exposure. The current baseline estimator operates on a homogeneous Markov transition and on the current-period monthly block; lagged memory and extreme-driven transition belong to the conceptual layer and to the forecasting or extension layer respectively.
