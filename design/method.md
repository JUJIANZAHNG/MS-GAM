# Chapter 2: Conceptual Framework after the MS-GAM Shift

Updated: 2026-06-28

## 1. Central view

Tourism demand is modeled as a regime-dependent functional system for each destination.

Let the observed climate state be:

[
X_t=(T_t,R_t,E_t)
]

and let \(\mathcal{M}_L(X_t)\) denote the climate-memory operator built from current and lagged climate information.

The core conceptual equation is:

[
\boxed{
Y_{r,t}
=
f_{r,S_{r,t}}
\left(
\mathcal{M}_L(X_t)
\right)
+
\varepsilon_{r,t}
}
]

This is the mother equation of the paper.

## 2. Why this is no longer a plain regression story

The conceptual object is not a fixed coefficient vector.
It is a destination-specific family of climate-response functions indexed by latent regime:

[
\mathbb{F}_r
=
\left\{
f_{r,1},\dots,f_{r,K_r}
\right\}
]

So each destination is treated as a dynamic system whose response surface can change across latent states.

## 3. Decomposition of the response function

At the empirical layer, each regime-specific function can be decomposed into:

[
f_{r,k}(X_t)
=
f_{r,k}^{temp}(T_t)
+
f_{r,k}^{rain}(R_t)
+
f_{r,k}^{int}(T_t,R_t)
+
f_{r,k}^{ext}(E_t)
]

where:

- \(f^{temp}\): nonlinear temperature effect
- \(f^{rain}\): nonlinear rainfall effect
- \(f^{int}\): continuous interaction surface
- \(f^{ext}\): explicit extreme-weather exposure

This separation matters because the interaction surface and the extreme term should stay conceptually distinct throughout the paper.

## 4. Climate memory operator

The system is not memoryless.

Define:

[
\mathcal{M}_L(X_t)
=
\left(
X_t,
X_{t-1},
\dots,
X_{t-L}
\right)
]

This operator is part of the conceptual foundation for the forecasting chapter, where lagged information and forecast-available climate blocks become explicit inputs.

## 5. Regime evolution

The regime process is:

[
S_{r,t}\in\{1,\dots,K_r\}
]

Conceptually, this means climate-response functions can switch across latent structural states over time:

[
f_{r,S_{r,t}}
\neq
f_{r,S_{r,t+1}}
\quad \text{in general}
]

This is the theoretical reason the paper moved from a single GAM to an MS-GAM design.

## 6. Functional heterogeneity

Different destinations are allowed to have different function families:

[
\mathbb{F}_r \neq \mathbb{F}_s
\quad \text{for } r\neq s
]

This gives the basis for Chapter 5, where comparison is made across regime-specific response objects rather than across pooled coefficients.

## 7. Conceptual role of extremes

The extreme-weather term \(E_t\) is part of the climate state, but its role is stronger than a generic covariate.

Conceptually, extreme conditions can:

| Channel | Meaning |
|---|---|
| direct exposure | enter the response function through \(f_{r,k}^{ext}(E_t)\) |
| interaction amplification | strengthen or reshape the temperature-rainfall surface |
| state perturbation | make regime switching more likely |

This is why Chapter 6 later tests whether extreme-aware and regime-aware information improves forecasting beyond regular climate blocks.

## 8. One-line summary

Chapter 2 can be summarized as:

> Tourism demand is modeled as a destination-specific, regime-dependent climate-response system defined over a memory-enhanced climate state, with explicit separation between regular climate effects, interaction structure, and extreme-weather exposure.
