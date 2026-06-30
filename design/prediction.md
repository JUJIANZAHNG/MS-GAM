# Chapter 6: Prediction under Regime and Extreme Climate Structure

Status: control file
Updated: 2026-06-30

## 1. Core role of Chapter 6

Chapter 6 is the forecasting and structural validation branch of the paper. It is not a GAM forecasting chapter. MS-GAM is the structural representation layer:

(y_{1:t}, X_{1:t}) → Π_t → Z_t

The downstream forecasting layer is:

ŷ_{t+1|t} = g_{θ*}(·)

Chapter 6 serves two linked purposes:

1. Practical usefulness: whether climate-aware, extreme-aware, and regime-aware information improves out-of-sample forecasting.
2. Structural validation: whether information extracted from MS-GAM has incremental predictive value beyond conventional autoregressive and climate-aware inputs.

The main focus is information gain, not a model horse race.

## 2. Forecast target and timing discipline

Let y_t = log(Y_t + 1). The core target is the one-step-ahead monthly forecast:

ŷ_{t+1|t}

Recursive multi-step forecasting (h ≥ 2) is a secondary extension. The one-step-ahead design is the core design.

Default forecast-side input blocks:

H_t = (y_t, ..., y_{t-L+1})
C_t = forecast-available regular-climate block at origin t
E_t = forecast-available extreme-weather block at origin t

If ex ante climate forecasts are not used, C_t and E_t reduce to lagged climate blocks. The same timing discipline must be imposed on the climate inputs of B_2 and M_2-M_4.

Forecast-availability rule:

- all inputs must be available at forecast origin t;
- B_2 and M_2-M_4 must obey the same climate timing discipline;
- if ex ante climate forecasts are later introduced, they must be added symmetrically across all climate-using models;
- only filtered regime information may enter forecasting.

## 3. Structural input from MS-GAM

At forecast origin t, the filtered regime-probability vector is:

Π_t = (π_{t,1}, ..., π_{t,K}),   π_{t,k} = P(S_t = k | y_{1:t}, X_{1:t})

Define the forecast-side structural block as:

Z_t = Ψ(Π_t) = (Π_t, H_t^Π, Ŝ_t)

where H_t^Π = -Σ_k π_{t,k} log π_{t,k} and Ŝ_t = argmax_k π_{t,k}.

This keeps the main M_4 input minimal. Additional MS-GAM summaries should be treated as robustness extensions rather than part of the baseline ladder.

Φ_r^{(k)} is the destination-specific functional or geometric summary of a regime-specific function used in Chapter 5. It is not a Chapter 6 baseline input. Chapter 6 consumes Z_t, not Φ_r^{(k)}.

## 4. External anchors

The external anchors are fixed reference points, not lower rungs in the same ablation sequence.

B_0:   ŷ_{t+1|t}^{B_0} = y_{t+1-12}                    (Seasonal Naive)
B_1:   ŷ_{t+1|t}^{B_1} = g^{SARIMA}(H_t)               (classical univariate benchmark)
B_2:   ŷ_{t+1|t}^{B_2} = g^{SARIMAX}(H_t, C_t)         (classical climate benchmark)

## 5. Information ablation ladder

Let G denote a small candidate set of downstream forecasting models. Select the forecasting engine once at the full-information rung:

g_{θ*} = argmin_{g ∈ G} CVErr_{M_4}(g)

using rolling-origin or time-series cross-validation. Once selected, the same g_{θ*} is held fixed across all M-series rungs. The ladder varies information, not model class.

M_1:   ŷ_{t+1|t}^{M_1} = g_{θ*}(H_t)
M_2:   ŷ_{t+1|t}^{M_2} = g_{θ*}(H_t, C_t)
M_3:   ŷ_{t+1|t}^{M_3} = g_{θ*}(H_t, C_t, E_t)
M_4:   ŷ_{t+1|t}^{M_4} = g_{θ*}(H_t, C_t, E_t, Z_t)

The increments correspond to:

demand history → regular climate → extreme-weather exposure → filtered regime structure.

## 6. Metric set

The main metric set is:

- MAE
- RMSE

These are computed on the one-step-ahead forecasts. R^2 is not part of the main metric set.

### 6.1 Relative gain metric

For two models M_a and M_b, define:

Gain(M_a, M_b) = (Err(M_b) - Err(M_a)) / Err(M_b)

where Err(·) is instantiated as MAE or RMSE. The numerator and denominator must use the same error measure within a single Gain value.

The Gain metric is reported separately for MAE and RMSE.

Key contrasts:

- Gain(M_2, M_1): incremental value of regular climate information
- Gain(M_3, M_2): incremental value of extreme-weather exposure
- Gain(M_4, M_3): incremental value of MS-GAM structural information — the primary structural-validation contrast
- Gain(M_4, B_1), Gain(M_4, B_2): practical forecasting usefulness against classical benchmarks

### 6.2 Primary structural-validation contrast

M_4 vs. M_3 is the primary structural-validation contrast. It holds the downstream learning engine and conventional inputs fixed and adds only the filtered regime structure Z_t. This contrast is the sharpest test of whether MS-GAM-extracted information has incremental predictive value.

## 7. Tail / extreme subset

The tail subset is the extreme-months subset defined by the extreme indicator E_t used by the baseline model.

Definition: the extreme-months subset is the set of months t for which the extreme indicator E_t = 1 under the destination-specific E_t construction used in the baseline MS-GAM estimator. The threshold τ used to construct E_t is the same destination-specific threshold used in the extreme-variable construction in `design/ms-gam-core.md` and `design/data.md`. No separate threshold is introduced for the tail-subset analysis.

The metrics in Section 6 are recomputed on this subset to produce tail-subset MAE, tail-subset RMSE, and tail-subset Gain values for the same key contrasts. The tail-subset analysis is a structural diagnostic of how the regime and extreme information performs in the most demanding subpopulation, not a separate model class.

## 8. Recursive multi-step extension

Multi-step forecasting is a recursive extension of the fitted one-step model. For h ≥ 2:

ŷ_{t+h|t}^{M_j} = g_{θ*}(Ĥ_{t+h-1|t}, I_t^{(j,h)})

where Ĥ updates the demand-history block with previously predicted values, and I_t^{(j,h)} denotes the forecast-available information path at forecast origin t.

In the main text, one-step-ahead forecasting remains the core design. Recursive multi-step is an extension used to examine how long the information gains persist beyond the immediate horizon.

## 9. Open items

The following items are not defined in this control file and are recorded as unresolved:

- The exact final E_t construction (binary vs. continuous vs. compound) is fixed in the empirical notebooks, not in this file.
- The exact final L, L_T, L_R, L_E values are fixed in the empirical notebooks.

## 10. Reporting discipline

The main tables should report:

- the fixed external anchors B_0, B_1, B_2;
- the information ablation ladder M_1 through M_4 under the same selected ML engine;
- MAE, RMSE, and Gain values for the key contrasts;
- tail-subset MAE, RMSE, and Gain values where applicable.

Machine-learning model screening belongs to a separate selection table or appendix, not to the main ladder table. If a second ML family is retained, it should be used only as a robustness check on the key contrasts, especially M_4 vs. M_3. This keeps Chapter 6 focused on information gain rather than a model horse race.
