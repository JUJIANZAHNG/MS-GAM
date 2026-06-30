# Chapter 4: Inference of Regime-Specific Climate-Response Functions

Status: control file
Updated: 2026-06-30

This file controls the Chapter 4 method note. The estimator and input block are specified in `design/ms-gam-core.md`; this file documents the estimation contract, the inference outputs, and the handoff to later chapters.

Important convention:

> All formulas are written for one destination. Different destinations are estimated separately, not as a pooled panel.

## 1. Role of Chapter 4

Chapter 4 is the inference bridge between the conceptual system in Chapter 2 and the comparative and predictive chapters that follow. Its job is to recover three objects from observed data:

| Object | Meaning | Used in |
|---|---|---|
| S_t | latent regime state | Chapters 5, 6, 7 |
| F^{(k)}(X_t) | regime-specific climate-response function | Chapters 5, 7 |
| Π_t | filtered posterior regime probabilities | Chapters 6, 7 |

Chapter 4 is not a "fit one GAM" section. It is the chapter where latent regimes and regime-specific smooth response functions are inferred jointly.

## 2. Observed data objects

Let monthly tourism demand be:

y_t = log(Y_t + 1)

Let the observed monthly block entering the baseline estimator be:

X_t = (T_t, R_t, E_t, m_t, t)

where:

- T_t: monthly temperature state
- R_t: monthly rainfall state
- E_t: extreme-weather exposure
- m_t: calendar month
- t: time index for long-run trend

This is the current-period monthly block. Lagged climate inputs do not enter the baseline. The data chapter is `design/data.md`.

## 3. Latent regime process

The latent regime is:

S_t ∈ {1, ..., K}

with first-order homogeneous Markov transition:

P(S_t = j | S_{t-1} = i) = p_{ij}

and transition matrix:

P = (p_{ij})_{K×K}

The transition matrix P is time-invariant under the baseline specification. Structural break analysis may be used as diagnostic evidence of regime instability, but the main regime object is the latent stochastic state S_t.

## 4. Regime-dependent observation equation

For each regime k, define the regime-specific smooth response function:

F^{(k)}(X_t) = s_k^{trend}(t) + s_k^{season}(m_t) + f_k^{temp}(T_t) + f_k^{rain}(R_t) + te_k(T_t, R_t) + f_k^{ext}(E_t)

The observation equation is:

y_t | S_t = k ~ N(F^{(k)}(X_t), σ_k^2)

Equivalently:

y_t = Σ_{k=1}^{K} 1(S_t = k) F^{(k)}(X_t) + ε_t

This makes Chapter 4 an inference problem over a family of smooth functions rather than a single destination-level response curve.

## 5. Joint likelihood

The joint model can be written as:

p(y_{1:T}, S_{1:T} | X_{1:T}) = p(S_1) Π_{t=2}^{T} P(S_t | S_{t-1}) Π_{t=1}^{T} p(y_t | S_t, X_t)

with:

p(y_t | S_t = k, X_t) = φ(y_t; F^{(k)}(X_t), σ_k^2)

where φ denotes the Gaussian density. This likelihood is the formal link that legitimizes the Markov-switching layer.

## 6. Estimation contract

The baseline estimator is the one specified in `design/ms-gam-core.md`. The estimation contract is:

### 6.1 Baseline inputs

The current-period monthly block X_t = (T_t, R_t, E_t, m_t, t). Lagged climate memory is not in the baseline input block.

### 6.2 Baseline transition type

First-order homogeneous Markov chain with constant transition matrix P. No climate covariate enters the transition equation.

### 6.3 Baseline observation equation

The regime-specific GAM given in `design/ms-gam-core.md`, Section 2.

### 6.4 Baseline outputs

The estimator delivers four outputs:

- Regime-specific smooth functions F̂^{(1)}, ..., F̂^{(K)}: the core objects for Chapter 5.
- Filtered regime probabilities Π_t: the forecast-safe structural inputs for Chapter 6.
- Smoothed regime probabilities P(S_t = k | y_{1:T}, X_{1:T}): used for interpretation, regime labeling, and structural diagnostics; not for the forecasting layer.
- Estimated transition matrix P̂: needed for regime persistence and transition analysis in Chapter 7.

### 6.5 Filtered vs. smoothed distinction

Two posterior summaries are produced:

- Filtered probabilities Π_t = (P(S_t = 1 | y_{1:t}, X_{1:t}), ..., P(S_t = K | y_{1:t}, X_{1:t})) use only information available at time t. Only these may enter the forecasting layer.
- Smoothed probabilities P(S_t = k | y_{1:T}, X_{1:T}) use the full sample including future observations. They are reserved for interpretation and structural diagnostics.

## 7. Practical estimation strategy

The implementation is an iterative inference scheme that alternates between regime inference and regime-specific function estimation. The exact algorithmic details (basis dimensions, penalty parameter values, initialization count, convergence criteria) are not fixed in this control file and should be specified in the empirical notebooks.

| Step | Input | Action | Output |
|---|---|---|---|
| 1 | K, initial state probabilities | initialize latent regimes | starting weights |
| 2 | current state weights | fit regime-weighted GAM components | F̂^{(k)} |
| 3 | fitted regime means and variances | update state probabilities and transition parameters | Π_t, P̂ |
| 4 | updated states | iterate until convergence | final regime-function system |

In exposition, it is enough to say that Chapter 4 jointly alternates between regime inference and regime-specific GAM smoothing. The chapter is not a full algorithm paper.

## 8. Open estimation parameters

The following items are not fixed in this control file and are recorded as unresolved:

- basis dimensions for each smooth term
- penalty parameter values
- initialization count and stopping rules
- specific implementation of the regime-weighted GAM fit

These should be fixed in the empirical notebooks and reported in the implementation appendix.

## 9. What Chapter 4 passes forward

| Chapter | Object handed off | Why it matters |
|---|---|---|
| Chapter 5 | F̂^{(k)} | compare regime-specific shapes and interaction geometry |
| Chapter 6 | Π_t and derived Z_t | build structure-aware forecasts |
| Chapter 7 | P̂, F̂^{(k)}, posterior regime paths | test structural robustness |

## 10. One-line closure

(y_{1:T}, X_{1:T}) ⇒ (F̂^{(1:K)}, P̂, Π_{1:T})

This is the inference layer of the paper. Later chapters must not fall back to a single, regime-free destination function.
