# Prediction Design under an Information Ladder

Updated: 2026-06-27

## 1. Core role of Chapter 6

Chapter 6 is not a GAM forecasting chapter.

MS-GAM is the structural representation layer:

[
(y_{1:t},X_{1:t})
\rightarrow
\Pi_t
\rightarrow
Z_t
]

The downstream forecasting layer is:

[
\hat{y}_{t+1\mid t}
=
g_{\theta^\star}(\cdot)
]

So the chapter serves two linked purposes:

1. practical usefulness: whether climate-aware, extreme-aware, and regime-aware information improves out-of-sample forecasting;
2. structural validation: whether information extracted from MS-GAM has incremental predictive value beyond conventional autoregressive and climate-aware inputs.

## 2. Forecast target and timing discipline

Let:

[
y_t=\log(Y_t+1)
]

The core target is the one-step-ahead monthly forecast:

[
\hat{y}_{t+1\mid t}
]

Define the default forecast-side input blocks:

[
\mathcal{H}_t
=
\left(
y_t,\dots,y_{t-L+1}
\right)
]

[
\mathcal{C}_t
=
\left(
T_t,\dots,T_{t-L_T+1},
R_t,\dots,R_{t-L_R+1}
\right)
]

[
\mathcal{E}_t
=
\left(
E_t,\dots,E_{t-L_E+1}
\right)
]

where seasonal lags such as \(y_{t-12}\) are treated as part of \(\mathcal{H}_t\).

If deterministic calendar features such as cyclical month encodings are used, they should enter all ML rungs uniformly and should not define a separate information rung.

Forecast-availability rule:

- all inputs must be available at forecast origin \(t\);
- \(B_2\) and \(M_2\)-\(M_4\) must obey the same climate timing discipline;
- if ex ante climate forecasts are later introduced, they must be added symmetrically across all climate-using models;
- only filtered regime information may enter forecasting.

## 3. Structural input from MS-GAM

At forecast origin \(t\), the filtered regime-probability vector is:

[
\Pi_t
=
\left(
\pi_{t,1},\dots,\pi_{t,K}
\right),
\qquad
\pi_{t,k}=P(S_t=k\mid y_{1:t},X_{1:t})
]

Define the forecast-side structural block as:

[
Z_t
=
\Psi(\Pi_t)
=
\left(
\Pi_t,
H_t^{\Pi},
\hat{S}_t
\right)
]

where

[
H_t^{\Pi}
=
-\sum_{k=1}^{K}\pi_{t,k}\log \pi_{t,k},
\qquad
\hat{S}_t=\arg\max_k \pi_{t,k}
]

This keeps the main \(M_4\) input minimal.
Additional MS-GAM summaries should be treated as robustness extensions rather than part of the baseline ladder.

## 4. External anchors

The external anchors are fixed references, not lower rungs in the same ablation sequence.

[
B_0:
\qquad
\hat{y}_{t+1\mid t}^{B_0}=y_{t+1-12}
]

Seasonal Naive.

[
B_1:
\qquad
\hat{y}_{t+1\mid t}^{B_1}
=
g^{SARIMA}(\mathcal{H}_t)
]

Classical univariate benchmark.

[
B_2:
\qquad
\hat{y}_{t+1\mid t}^{B_2}
=
g^{SARIMAX}(\mathcal{H}_t,\mathcal{C}_t)
]

Classical climate benchmark.

## 5. Information ablation ladder

Let \(\mathcal{G}\) denote a small candidate set of downstream forecasting models, for example XGBoost, LightGBM, MLP, or a similarly compact set.
Select the forecasting engine once at the full-information rung:

[
g_{\theta^\star}
=
\arg\min_{g\in\mathcal{G}}
CVErr_{M_4}(g)
]

using rolling-origin or time-series cross-validation.

Once selected, the same \(g_{\theta^\star}\) is held fixed across all ML rungs.
The ladder then varies information, not model class.

[
M_1:
\qquad
\hat{y}_{t+1\mid t}^{M_1}
=
g_{\theta^\star}(\mathcal{H}_t)
]

[
M_2:
\qquad
\hat{y}_{t+1\mid t}^{M_2}
=
g_{\theta^\star}(\mathcal{H}_t,\mathcal{C}_t)
]

[
M_3:
\qquad
\hat{y}_{t+1\mid t}^{M_3}
=
g_{\theta^\star}(\mathcal{H}_t,\mathcal{C}_t,\mathcal{E}_t)
]

[
M_4:
\qquad
\hat{y}_{t+1\mid t}^{M_4}
=
g_{\theta^\star}(\mathcal{H}_t,\mathcal{C}_t,\mathcal{E}_t,Z_t)
]

The increments correspond to:

[
\text{demand history}
\rightarrow
\text{regular climate}
\rightarrow
\text{extreme-weather exposure}
\rightarrow
\text{filtered regime structure}
]

## 6. Interpretation of contrasts

The chapter should read the ladder through four contrasts:

[
B_1,\ B_2
\quad
\text{vs.}
\quad
M_4
]

practical forecasting usefulness against classical benchmarks.

[
M_2
\quad
\text{vs.}
\quad
M_1
]

incremental value of regular climate information.

[
M_3
\quad
\text{vs.}
\quad
M_2
]

incremental value of extreme-weather exposure beyond regular climate.

[
M_4
\quad
\text{vs.}
\quad
M_3
]

incremental value of MS-GAM structural information.

The \(M_4\) versus \(M_3\) contrast is the primary structural-validation test.
It holds the forecasting engine and conventional inputs fixed and adds only filtered regime structure.

## 7. Recursive multi-step extension

Multi-step forecasting is treated as a recursive extension of the fitted one-step model.

For \(h\geq 2\),

[
\hat{y}_{t+h\mid t}^{M_j}
=
g_{\theta^\star}
\left(
\widehat{\mathcal{H}}_{t+h-1\mid t},
\mathcal{I}_{t}^{(j,h)}
\right)
]

where \(\widehat{\mathcal{H}}_{t+h-1\mid t}\) updates the demand-history block with previously predicted values, and

[
\mathcal{I}_{t}^{(1,h)}=\varnothing,
\qquad
\mathcal{I}_{t}^{(2,h)}=\mathcal{C}_{t}^{(h)},
\qquad
\mathcal{I}_{t}^{(3,h)}=\left(\mathcal{C}_{t}^{(h)},\mathcal{E}_{t}^{(h)}\right),
\qquad
\mathcal{I}_{t}^{(4,h)}=\left(\mathcal{C}_{t}^{(h)},\mathcal{E}_{t}^{(h)},Z_t\right)
]

denotes the forecast-available information path at forecast origin \(t\).

In the main text, one-step-ahead forecasting remains the core design.
Recursive multi-step forecasting is an extension used to examine how long the information gains persist beyond the immediate horizon.

## 8. Reporting discipline

The main tables should report:

- the fixed external anchors \(B_0,B_1,B_2\);
- the information ablation ladder \(M_1\) through \(M_4\) under the same selected ML engine.

Machine-learning model screening belongs to a separate selection table or appendix, not to the main ladder table.
If a second ML family is retained, it should be used only as a robustness check on the key contrasts, especially \(M_3\) versus \(M_4\).

This keeps Chapter 6 focused on information gain rather than a model horse race.
