# Chapter 7: Structural Robustness after the MS-GAM Shift

Updated: 2026-06-28

This file controls the robustness chapter after the move from a single-function GAM paper to a regime-dependent functional system paper.

## 1. Role of Chapter 7

Chapter 7 is not a generic appendix of extra checks.

Its job is to test whether the main structural objects of the paper remain credible under perturbations of data, sample design, and specification.

The three robustness targets are:

| Target | Object | Question |
|---|---|---|
| regime stability | posterior regime path and transition dynamics | does the latent-state structure survive specification changes? |
| function stability | \(\hat{\mathcal{F}}_r^{(k)}\) | do regime-specific response functions remain similar? |
| prediction-gain stability | \(M_4\) versus \(M_3\) and related contrasts | does structural information keep adding forecast value? |

So the chapter is a structural robustness chapter, not a coefficient-robustness chapter.

## 2. Regime stability

The first object to test is the inferred regime structure itself.

Key quantities:

[
P(S_t=k\mid y_{1:T},X_{1:T})
]

[
\hat{\mathbf{P}}=(\hat p_{ij})_{K\times K}
]

Useful perturbations:

| Perturbation | Why it matters |
|---|---|
| alternative \(K\) | checks whether regime count is arbitrary |
| alternative smoothing penalties / basis size | checks whether state inference is driven by over-flexible functions |
| alternative \(E_t\) construction | checks whether regime structure depends too heavily on one extreme definition |
| alternative sample windows | checks whether states collapse when the sample boundary changes |

The chapter should look for whether posterior regime paths, state persistence, and broad transition timing remain qualitatively similar.

## 3. Function stability within regime

The second object is the regime-specific function family:

[
\hat{\mathcal{F}}_r^{(k)}
=
\left(
\hat f_{r,k}^{temp},
\hat f_{r,k}^{rain},
\widehat{te}_{r,k},
\hat f_{r,k}^{ext}
\right)
]

The basic robustness question is:

[
\hat{\mathcal{F}}_{r,\text{base}}^{(k)}
\approx
\hat{\mathcal{F}}_{r,\text{alt}}^{(k)}
]

This does not require pointwise identity. It requires stability of economically meaningful shape features:

| Feature | What should stay stable |
|---|---|
| marginal temperature effect | sign pattern, threshold region, strong bends |
| marginal rainfall effect | overall slope shape and nonlinear turning areas |
| interaction surface | whether coupling is weak, moderate, or strong |
| extreme component | whether explicit extreme exposure remains salient or negligible |

This is the place to test whether Chapter 5 geometry is a stable object rather than a specification artifact.

## 4. Geometry stability

Using the feature map from Chapter 5:

[
\Phi_r^{(k)}
=
\left(
C_{r,k}^{temp},
C_{r,k}^{rain},
G_{r,k}^{int},
A_{r,k}^{ext}
\right)
]

test whether:

[
\Phi_{r,base}^{(k)}
\approx
\Phi_{r,alt}^{(k)}
]

Practical checks:

| Check | Output |
|---|---|
| alternative smoothing / basis | summary-feature comparison table |
| bootstrap or resampled refit | dispersion band for geometry summaries |
| alternative regime matching rule | cluster and distance stability |

This keeps Chapter 7 tied to the same regime-specific objects used in Chapter 5.

## 5. Aggregation and frequency invariance

The paper is monthly in its main design, but some destinations have daily data that can support frequency checks.

The clean use of daily data here is:

| Destination | What daily data support |
|---|---|
| Macao | daily-to-monthly climate aggregation checks and monthly consistency of extreme indicators |
| Jiuzhaigou | daily flow aggregation checks, daily climate consistency, and month-completeness sensitivity |

> **Source of truth.** The list of destinations with daily support is fixed in `design/data.md` §3.2. The current list above reflects the destinations for which daily → monthly aggregation has been verified. If `design/data.md` adds or removes a destination, this table must be updated to match before Chapter 7 is finalized. Do not let the two files drift.

The point is not to turn the main paper into a daily forecasting paper.
The point is to ask whether the monthly structural conclusions are sensitive to the way daily information is aggregated into monthly climate or tourism measures.

## 6. Prediction-gain stability

Chapter 6 establishes the main contrast:

[
M_4
\quad \text{vs.} \quad
M_3
]

Chapter 7 asks whether that gain survives perturbations.

Define:

[
Gain_{structure}
=
\frac{Err(M_3)-Err(M_4)}{Err(M_3)}
]

The exact error metric \(Err(\cdot)\) can later be instantiated as MAE or RMSE, but the robustness logic is already clear:

> structural information matters only if the gain stays positive beyond one convenient specification.

Useful checks:

| Check | Purpose |
|---|---|
| rolling-origin split changes | stability across evaluation windows |
| extreme-month subset | whether gain concentrates in stressed periods |
| normal-month subset | whether gain disappears outside stressed periods |
| alternative downstream ML engine | whether the structural contribution is model-specific |

The key point is to test gain stability, not just absolute forecast accuracy.

## 7. What counts as failure

The robustness chapter should state failure conditions clearly.

| Failure type | Meaning |
|---|---|
| regime collapse | inferred regimes disappear when mild specification changes are introduced |
| geometry reversal | main shape and interaction conclusions flip across reasonable alternatives |
| gain disappearance | \(M_4\) loses its edge once evaluation becomes slightly stricter |

Without explicit failure conditions, the chapter turns into a list of extra tables rather than a real robustness argument.

## 8. One-line closure

Chapter 7 can be summarized as:

[
\boxed{
\left(
\hat S_{r,t},
\hat{\mathcal{F}}_r^{(k)},
Gain_{structure}
\right)
\text{ should remain qualitatively stable under reasonable perturbations}
}
]

This is the final check that the paper is recovering a durable structural system rather than a fragile modeling artifact.

