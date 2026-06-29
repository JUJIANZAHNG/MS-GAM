# Chapter 6 Forecast Evaluation Brief

Updated: 2026-06-28

## 1. Chapter role

Chapter 6 serves two purposes at the same time:

1. forecasting usefulness: whether additional climate, extreme, and structural information improves out-of-sample prediction;
2. structural validation: whether information extracted from the MS-GAM system has incremental predictive value beyond conventional inputs.

The chapter is therefore not a plain forecast horse race.

## 2. Paper-level context

The paper is built around separate destination-level estimation.
It is not a pooled panel design.

For each destination, the main structural model is a Markov-Switching GAM:

[
S_t \in \{1,\dots,K\}
]

[
y_t \mid S_t=k
\sim
\mathcal{N}
\left(
\mathcal{F}^{(k)}(X_t),
\sigma_k^2
\right)
]

with:

[
\mathcal{F}^{(k)}(X_t)
=
s_k^{trend}(t)
+
s_k^{season}(m_t)
+
f_k^{temp}(T_t)
+
f_k^{rain}(R_t)
+
te_k(T_t,R_t)
+
f_k^{ext}(E_t)
]

The main monthly outcome is:

[
y_t=\log(Y_t+1)
]

Daily data mainly support extreme-weather construction, monthly aggregation checks, and robustness.
The main forecasting target is still monthly tourism demand.

## 3. Current Chapter 6 working design

### 3.1 Forecast target

The current core target is one-step-ahead monthly forecasting:

[
\hat y_{t+1\mid t}
]

Recursive multi-step forecasting is currently treated as an extension rather than the main design.

### 3.2 Structural input from MS-GAM

At forecast origin \(t\), the filtered regime probabilities are:

[
\Pi_t
=
\left(
\pi_{t,1},\dots,\pi_{t,K}
\right),
\qquad
\pi_{t,k}=P(S_t=k\mid y_{1:t},X_{1:t})
]

The current structural block is:

[
Z_t
=
\left(
\Pi_t,
H_t^{\Pi},
\hat S_t
\right)
]

where:

[
H_t^{\Pi}
=
-\sum_{k=1}^{K}\pi_{t,k}\log \pi_{t,k},
\qquad
\hat S_t=\arg\max_k \pi_{t,k}
]

Only filtered regime information is allowed to enter forecasting.
Smoothed posterior probabilities are excluded from the forecast layer.

### 3.3 Current information ladder

External anchors:

[
B_0:
\qquad
\hat y_{t+1\mid t}^{B_0}=y_{t+1-12}
]

[
B_1:
\qquad
\hat y_{t+1\mid t}^{B_1}
=
g^{SARIMA}(\mathcal H_t)
]

[
B_2:
\qquad
\hat y_{t+1\mid t}^{B_2}
=
g^{SARIMAX}(\mathcal H_t,\mathcal C_t)
]

ML-style information ladder:

[
M_1:
\qquad
\hat y_{t+1\mid t}^{M_1}
=
g_{\theta}(\mathcal H_t)
]

[
M_2:
\qquad
\hat y_{t+1\mid t}^{M_2}
=
g_{\theta}(\mathcal H_t,\mathcal C_t)
]

[
M_3:
\qquad
\hat y_{t+1\mid t}^{M_3}
=
g_{\theta}(\mathcal H_t,\mathcal C_t,\mathcal E_t)
]

[
M_4:
\qquad
\hat y_{t+1\mid t}^{M_4}
=
g_{\theta}(\mathcal H_t,\mathcal C_t,\mathcal E_t,Z_t)
]

The intended interpretation is:

[
\text{demand history}
\rightarrow
\text{regular climate}
\rightarrow
\text{extreme-weather exposure}
\rightarrow
\text{filtered regime structure}
]

## 4. Current input-block notation

The current working notation is:

[
\mathcal H_t
=
\left(
y_t,\dots,y_{t-L+1}
\right)
]

[
\mathcal C_t
=
\left(
T_t,\dots,T_{t-L_T+1},
R_t,\dots,R_{t-L_R+1}
\right)
]

[
\mathcal E_t
=
\left(
E_t,\dots,E_{t-L_E+1}
\right)
]

Seasonal lags such as \(y_{t-12}\) are currently treated as part of \(\mathcal H_t\).
If cyclical month encodings are used, they are intended to enter all ML rungs uniformly rather than define a separate rung.

## 5. Constraints that any final design must respect

| Constraint | Meaning |
|---|---|
| no leakage | all forecast inputs must be available at forecast origin \(t\) |
| single-destination estimation | forecasting design should not assume pooled panel information |
| monthly main target | daily information can support, but should not silently change the main prediction unit |
| structural interpretability | the ladder should still allow a clear reading of climate gain, extreme gain, and regime-structure gain |
| small/medium sample realism | lag design and model complexity must fit destination-level monthly samples |
| comparability | climate-using benchmarks and ML models should obey the same timing discipline |

## 6. Components that are still unresolved

### 6.1 Error metric \(Err(\cdot)\)

The symbol \(Err(\cdot)\) has been left abstract.
Candidate metrics under discussion include MAE, RMSE, and other scale-sensitive or scale-free forecast metrics.

The main unresolved issue is not just which metric to use, but how to layer:

- main point-forecast metrics
- tail / extreme-period metrics
- incremental gain metrics
- possible statistical comparison tests

### 6.2 Tail subset definition

The tail / extreme subset has not been fixed.
Possible constructions include:

- thresholding the extreme-weather indicator \(E_t\)
- top quantiles of monthly climate stress
- externally defined event months
- response-side tail months

The subset definition matters because Chapter 6 wants to say something about predictive usefulness under stress, not only in average months.

### 6.3 Lag structure

The lag lengths for \(\mathcal H_t\), \(\mathcal C_t\), and \(\mathcal E_t\) are not yet fixed.

Open possibilities include:

- short autoregressive memory only
- seasonal memory plus short lags
- longer climate memory than demand memory
- different lag lengths for regular climate and extreme indicators

### 6.4 Timing of climate variables

One unresolved issue is whether climate inputs in forecasting should be:

- lagged only;
- contemporaneous if treated as forecast-available;
- or supported by explicit ex ante climate forecasts.

This choice affects both interpretability and leakage risk.

### 6.5 Model-screening rule

The current working idea is to screen a small candidate set of downstream ML models and then hold one learning engine fixed across \(M_1\) to \(M_4\).

This rule has not been fully locked.
Possible alternatives include:

- one fixed engine for the main ladder;
- several engines in the main chapter;
- one main engine plus one or two robustness engines.

## 7. What the final Chapter 6 design needs to achieve

Any final design should answer these four questions cleanly:

1. What is the main forecast target and forecast horizon?
2. What exact metric system defines forecast performance and incremental gain?
3. What exact information blocks enter each model, with what lag structure and timing discipline?
4. Does the final ladder support both practical usefulness and structural validation without becoming a model horse race?
